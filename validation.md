# Java Input Validation Guidelines

## 1. Null Validation

- ✅ Validate null inputs in public APIs and constructors
- ✅ Use explicit null checks with clear error messages
```java
// Good - Explicit null check
public void processUser(User user) {
    Objects.requireNonNull(user, "User cannot be null");
    // Process user
}

// Good - Annotation-based validation
public void saveUser(@NotNull User user) {
    // With Bean Validation or Spring Validation
    // Process user
}

// Bad - No validation
public void processUser(User user) {
    String name = user.getName(); // Potential NPE
}
```

## 2. Input Validation

- ✅ Reject invalid data (empty strings, negative values)
- ✅ Validate early and fail fast
```java
// Good - Comprehensive validation
public void transferMoney(Account source, Account target, BigDecimal amount) {
    Objects.requireNonNull(source, "Source account cannot be null");
    Objects.requireNonNull(target, "Target account cannot be null");
    Objects.requireNonNull(amount, "Amount cannot be null");
    
    if (amount.compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("Amount must be positive");
    }
    
    if (source.getBalance().compareTo(amount) < 0) {
        throw new InsufficientFundsException("Insufficient funds in source account");
    }
    
    // Proceed with transfer
}
```

## 3. Boundary Validation

- ✅ Check array/collection bounds before access
- ✅ Validate ranges and limits
```java
// Good - Boundary checks
public String getItem(List<String> items, int index) {
    if (items == null || items.isEmpty()) {
        throw new IllegalArgumentException("Items list cannot be null or empty");
    }
    
    if (index < 0 || index >= items.size()) {
        throw new IndexOutOfBoundsException("Index " + index + 
            " out of bounds for size " + items.size());
    }
    
    return items.get(index);
}

// Bad - No boundary checks
public String getItem(List<String> items, int index) {
    return items.get(index); // Potential IndexOutOfBoundsException
}
```

## 4. Bean Validation

- ✅ Use Bean Validation annotations for declarative validation
- ✅ Create custom validators for complex rules
```java
// Good - Bean Validation
public class User {
    @NotNull
    @Size(min = 2, max = 50)
    private String name;
    
    @NotNull
    @Email
    private String email;
    
    @NotNull
    @Pattern(regexp = "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{8,}$", 
             message = "Password must be at least 8 characters with letters and numbers")
    private String password;
    
    // Getters and setters
}

// Usage with validation
@RestController
public class UserController {
    @PostMapping("/users")
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        // If validation fails, MethodArgumentNotValidException is thrown
        return ResponseEntity.ok(userService.createUser(user));
    }
}
```

## 5. Defensive Programming

- ✅ Validate preconditions before operations
- ✅ Check state invariants
```java
// Good - Precondition checks
public class OrderProcessor {
    private boolean initialized = false;
    
    public void initialize() {
        // Initialization logic
        initialized = true;
    }
    
    public void processOrder(Order order) {
        if (!initialized) {
            throw new IllegalStateException("OrderProcessor not initialized");
        }
        
        Objects.requireNonNull(order, "Order cannot be null");
        
        // Process order
    }
}
```

## 6. Validation Best Practices

- ✅ Provide clear error messages with context
- ✅ Validate at system boundaries (API endpoints, external interfaces)
- ✅ Use domain-specific exceptions for validation failures
```java
// Good - Domain-specific exceptions
public class AccountService {
    public void withdraw(String accountId, BigDecimal amount) {
        if (accountId == null || accountId.trim().isEmpty()) {
            throw new InvalidAccountException("Account ID cannot be null or empty");
        }
        
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new InvalidAmountException("Amount must be positive");
        }
        
        Account account = accountRepository.findById(accountId)
            .orElseThrow(() -> new AccountNotFoundException("Account not found: " + accountId));
            
        if (account.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException(
                "Insufficient funds: available=" + account.getBalance() + 
                ", requested=" + amount);
        }
        
        // Proceed with withdrawal
    }
} 