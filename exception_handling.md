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

## 3. Layer-Specific Exception Handling

### DAO Layer
- ✅ Always throw exceptions, never handle them silently
- ✅ Translate database exceptions to domain exceptions
- ✅ Include entity information in exception messages

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
- ✅ Add business context to exceptions

```java
// Good - Service layer
public void processOrder(Order order) {
    try {
        validateOrder(order);
        orderDao.save(order);
        notifyShipping(order);
    } catch (ValidationException e) {
        // Handle validation failure specifically
        throw new BusinessException("Order validation failed", e);
    } catch (ResourceNotFoundException e) {
        // Handle not found specifically
        throw new BusinessException("Required resource not found", e);
    }
}
```

### REST Layer
- ✅ Let global exception handlers manage exceptions
- ✅ Add request context to exceptions when needed
- ✅ Don't catch exceptions unless adding value

```java
// Good - REST layer
@PostMapping("/orders")
public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody OrderRequest request) {
    // Let exceptions propagate to global handler
    Order order = orderService.createOrder(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(mapper.toResponse(order));
}

// Bad - REST layer
@PostMapping("/orders")
public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody OrderRequest request) {
    try {
        Order order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(mapper.toResponse(order));
    } catch (Exception e) {
        // Don't handle exceptions here
        log.error("Error creating order", e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
    }
}
```

## 4. Exception Catching

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
```

## 5. Resource Management

```java
// Good
try (Connection conn = dataSource.getConnection()) {
    // Use connection
} catch (SQLException e) {
    throw new BusinessException("Database error", e);
}
```

## 6. Global Exception Handling

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
} 