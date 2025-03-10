# Java Exception Handling Guidelines

## 1. Key Principles

- ✅ Use domain-specific exceptions like `BusinessException` instead of generic ones
- ✅ Catch specific exceptions, not `Exception` or `Throwable`
- ✅ Always use try-with-resources for closeable resources
- ✅ Either log OR throw (not both) at the same level
- ✅ Include context in exception messages

## 2. Enterprise Exception Structure

### Company Domain Exceptions
```java
// Always use the company exceptions
BusinessException           // For business logic errors
ResourceNotFoundException   // For resource not found errors
DuplicateResourceException  // For duplicate resource errors
CheckedBusinessException    // For checked business exceptions

// Example with BusinessException
throw new BusinessException("Invalid order state", "ERR-1001");

// With GUI message and error code
BusinessException ex = new BusinessException("System message");
ex.setGuiMessage("User-friendly message");
ex.setErrCode("ERR-1001");
throw ex;

// With dynamic parameters
Map<String, String> params = new HashMap<>();
params.put("orderId", "12345");
throw new BusinessException("Order processing failed", params);
```

### Custom Domain Exceptions
- ✅ Don't limit yourself to just the provided domain exceptions
- ✅ Create specific exceptions based on business scenarios when needed
- ✅ Extend the base domain exceptions for consistency
- ✅ Core exceptions come from the core jar, but create microservice-specific exceptions in your microservice package

```java
// Microservice-specific exceptions (create in your microservice package)
com.enttribe.exception.payment.InsufficientFundsException
com.enttribe.exception.order.OrderAlreadyProcessedException
com.enttribe.exception.user.UserLockedException
```

```java
// Good - Creating specific business exceptions
package com.enttribe.exception.payment;

public class InsufficientFundsException extends BusinessException {
    public InsufficientFundsException(String accountId, BigDecimal requested, BigDecimal available) {
        super("Insufficient funds in account: " + accountId);
        setGuiMessage("Your account doesn't have sufficient funds for this transaction");
        setErrCode("ERR-FUNDS-1001");
        
        Map<String, String> params = new HashMap<>();
        params.put("accountId", accountId);
        params.put("requested", requested.toString());
        params.put("available", available.toString());
        setParams(params);
    }
}

// Usage
if (account.getBalance().compareTo(withdrawalAmount) < 0) {
    throw new InsufficientFundsException(
        account.getId(), 
        withdrawalAmount, 
        account.getBalance()
    );
}
```

## 3. Layer-Specific Exception Handling

### DAO Layer
- ✅ Always throw exceptions, never handle them silently
- ✅ Translate database exceptions to domain exceptions
- ✅ Include entity information in exception messages as well as in log messages

```java
// Good - DAO layer
public User findById(Long id) {
    try {
        return jdbcTemplate.queryForObject(SQL, User.class);
    } catch (DataAccessException e) {
        throw new ResourceNotFoundException("User not found with ID: " + id, e);
    }
}

// Bad - DAO layer
public User findById(Long id) {
    try {
        return jdbcTemplate.queryForObject(SQL, User.class);
    } catch (DataAccessException e) {
        log.error("Error finding user", e);
        return null; // Don't return null or default values
    }
}
```

### Service Layer
- ✅ Handle exceptions based on business cases
- ✅ Translate technical exceptions to business exceptions
- ✅ Add business context to exceptions as well as in log messages
- ✅ Use custom domain exceptions where required, not generic ones

```java
// Good - Service layer with custom exceptions
public void processOrder(Order order) {
    try {
        validateOrder(order);
        orderDao.save(order);
        notifyShipping(order);
    } catch (ValidationException e) {
        // Using custom domain exception
        throw new OrderValidationException("Order validation failed for ID: " + order.getId(), e);
    } catch (ResourceNotFoundException e) {
        // Using custom domain exception with context
        throw new OrderDependencyException("Required resource not found for order: " + order.getId(), e);
    } catch (DataAccessException e) {
        // Using custom domain exception for data access issues
        throw new OrderPersistenceException("Failed to persist order: " + order.getId(), e);
    }
}

// Bad - Service layer with generic exceptions
public void processOrder(Order order) {
    try {
        validateOrder(order);
        orderDao.save(order);
        notifyShipping(order);
    } catch (ValidationException e) {
        // Using generic BusinessException instead of specific one
        throw new BusinessException("Order validation failed", e);
    } catch (ResourceNotFoundException e) {
        // Using generic BusinessException instead of specific one
        throw new BusinessException("Required resource not found", e);
    }
}

// Bad - Service layer
public void processOrder(Order order) {
    try {
        validateOrder(order);
        orderDao.save(order);
        notifyShipping(order);
    } catch (Exception e) {
        // Too generic exception handling
        log.error("Error processing order", e);
        // Swallowing the exception or returning null
        return;
    }
}

// Bad - Service layer
public void processOrder(Order order) {
    try {
        validateOrder(order);
        orderDao.save(order);
        notifyShipping(order);
    } catch (Exception e) {
        // Both logging AND throwing - pick one
        log.error("Error processing order", e);
        throw new RuntimeException("Order processing failed", e);
    }
}
```

### REST Layer
- ✅ Let global exception handlers manage exceptions
- ✅ Add request context to exceptions when needed
- ✅ Don't catch exceptions unless adding value
- ✅ Throw specific custom domain exceptions when validation fails at the controller level

```java
// Good - REST layer with custom exceptions if needed specifically for the controller
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final OrderService orderService;
    
    @PostMapping
    public ResponseEntity<ResponseWrapper<OrderDTO>> createOrder(@Valid @RequestBody OrderRequest request) {
        // Validation that can only be done at controller level
        if (request.getDeliveryDate().isBefore(LocalDate.now())) {
            throw new InvalidOrderDateException("Delivery date cannot be in the past", request.getDeliveryDate());
        }
        
        // Let service exceptions propagate to global handler
        OrderDTO order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(new ResponseWrapper<>(order));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ResponseWrapper<OrderDTO>> getOrder(@PathVariable String id) {
        // Validate ID format if needed
        if (!id.matches("^ORD-[0-9]{6}$")) {
            throw new InvalidOrderIdFormatException("Invalid order ID format: " + id);
        }
        
        OrderDTO order = orderService.findById(id);
        return ResponseEntity.ok(new ResponseWrapper<>(order));
    }
}

// Bad - REST layer with generic exceptions
@PostMapping("/orders")
public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody OrderRequest request) {
    // Using generic exception instead of specific one
    if (request.getDeliveryDate().isBefore(LocalDate.now())) {
        throw new BusinessException("Invalid delivery date");
    }
    
    Order order = orderService.createOrder(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(mapper.toResponse(order));
}
```

## 4. Exception Catching
This is the recommended way to catch exceptions. Do not catch `Exception` or `Throwable` directly instead catch the specific exception.

```java
// Good
try {
    processData();
} catch (ResourceNotFoundException e) {
    // Handle resource not found scenario
} catch (DuplicateResourceException e) {
    // Handle duplicate resource scenario
} catch (BusinessException e) {
    // Handle business exception
}

// Bad
try {
    processData();
} catch (Exception e) {  // Too generic
    log.error("Error", e);
}


## 5. Global Exception Handling

The application uses `RestExceptionHandler` to centrally handle all exceptions.

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(e.getErrCode(), e.getGuiMessage()));
    }
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusiness(BusinessException e) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse(e.getErrCode(), e.getGuiMessage()));
    }
    
    @ExceptionHandler(InvalidOrderDateException.class)
    public ResponseEntity<ErrorResponse> handleInvalidOrderDate(InvalidOrderDateException e) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse(e.getErrCode(), e.getGuiMessage()));
    }
    
    @ExceptionHandler(InvalidOrderIdFormatException.class)
    public ResponseEntity<ErrorResponse> handleInvalidOrderIdFormat(InvalidOrderIdFormatException e) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse(e.getErrCode(), e.getGuiMessage()));
    }
} 
```

## 6. Exception Flow from Service to REST Response

### Service to REST Exception Propagation
- ✅ Service layer exceptions should propagate naturally to the REST layer
- ✅ REST controllers should not catch service exceptions (unless adding context)
- ✅ Global exception handlers will automatically convert exceptions to appropriate HTTP responses

```java
// Service layer throws custom domain exception
@Service
public class OrderServiceImpl implements OrderService {
    @Override
    public OrderDTO createOrder(OrderRequest request) {
        // Business validation
        if (!inventoryService.hasStock(request.getProductId(), request.getQuantity())) {
            throw new InsufficientInventoryException(
                request.getProductId(), 
                request.getQuantity()
            );
        }
        
        // More processing...
    }
}

// REST controller doesn't catch the exception
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final OrderService orderService;
    
    @PostMapping
    public ResponseEntity<ResponseWrapper<OrderDTO>> createOrder(@Valid @RequestBody OrderRequest request) {
        // Service exceptions will propagate to global handler
        OrderDTO order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(new ResponseWrapper<>(order));
    }
}

// Global exception handler manages the response
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(InsufficientInventoryException.class)
    public ResponseEntity<ErrorResponse> handleInsufficientInventory(InsufficientInventoryException e) {
        // Log if needed
        log.warn("Insufficient inventory: {}", e.getMessage());
        
        // Return appropriate response
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .body(new ErrorResponse(
                e.getErrCode(), 
                e.getGuiMessage(),
                e.getParams()
            ));
    }
}
```

### Exception Flow Diagram

```
Service Layer                  REST Layer                    Global Handler
┌─────────────┐                ┌─────────────┐               ┌─────────────┐
│             │                │             │               │             │
│  throw new  │                │ Controller  │               │ @Exception  │
│ CustomExcep │───Propagate───▶│ doesn't     │───Capture────▶│ Handler     │
│    tion     │                │ catch       │               │ methods     │
│             │                │             │               │             │
└─────────────┘                └─────────────┘               └──────┬──────┘
                                                                    │
                                                                    │
                                                                    ▼
                                                            ┌─────────────┐
                                                            │ HTTP        │
                                                            │ Response    │
                                                            │ with error  │
                                                            │ details     │
                                                            └─────────────┘
```

### Benefits of This Approach
- ✅ Clean separation of concerns
- ✅ Centralized exception handling logic
- ✅ Consistent error responses across the API
- ✅ Service layer can focus on business logic without HTTP concerns
- ✅ Controllers remain thin and focused on request/response handling