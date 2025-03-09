# Java Naming and Style Guidelines

## 1. Naming Conventions

### Classes and Interfaces
- ✅ Use PascalCase
- ✅ Use nouns or noun phrases
- ✅ Be descriptive and avoid abbreviations
- ✅ Remove "I" prefix from interfaces
```java
// Good
public class UserService
public class OrderProcessor
public interface PaymentGateway

// Bad
public class UsrSvc
public class Processor
public interface IPaymentGateway
```

### Methods
- ✅ Use camelCase
- ✅ Start with verbs
- ✅ Be descriptive of the action
- ✅ Follow common prefixes: get/set/is/has/should
```java
// Good
public void processOrder(Order order)
public User findUserById(Long id)
public boolean isValidPayment(Payment payment)
public boolean hasPermission(String role)

// Bad
public void order(Order o)
public User byId(Long id)
public boolean checkIfValid(Payment p)
```

### Variables
- ✅ Use camelCase
- ✅ Be meaningful and descriptive
- ✅ Collections should indicate plurality
- ✅ Boolean variables should ask a question
```java
// Good
private final UserRepository userRepository;
private List<Order> pendingOrders;
private boolean isActive;
private boolean hasPermission;
for (int i = 0; i < items.length; i++) // i is OK in loops

// Bad
private final UserRepository repo;
private List<Order> order;
private boolean active;
private boolean permission;
```

### Constants
- ✅ Use UPPER_SNAKE_CASE
- ✅ Include units in name when applicable
- ✅ Group related constants in enum
```java
// Good
private static final int MAX_RETRY_ATTEMPTS = 3;
private static final long CACHE_EXPIRY_SECONDS = 3600;
private static final String API_BASE_URL = "https://api.example.com";

enum TimeUnit {
    DAYS,
    HOURS,
    MINUTES
}

// Bad
private static final int MAX = 3;
private static final long EXPIRY = 3600;
```

### Packages
- ✅ Use lowercase
- ✅ Use reverse domain name convention
- ✅ Use meaningful hierarchy
```java
// Good
com.company.module.feature
com.company.order.service
com.company.user.repository

// Bad
com.company.stuff
com.company.misc
```

## 2. Code Organization

### File Structure
1. Package declaration
2. Imports (ordered: java.*, javax.*, org.*, com.*)
3. Class/Interface declaration
4. Static fields
5. Instance fields
6. Constructors
7. Public methods
8. Protected methods
9. Private methods
10. Inner classes/interfaces

### Method Organization
- ✅ Maximum 30 lines per method
- ✅ Single responsibility principle
- ✅ Related methods should be grouped
- ✅ Override methods should be kept together

## 3. Code Style

### Formatting
- ✅ Use 4 spaces for indentation (not tabs)
- ✅ Maximum line length: 120 characters
- ✅ One blank line between methods
- ✅ No trailing whitespace
- ✅ Always use braces, even for single-line blocks
```java
// Good
if (condition) {
    doSomething();
}

// Bad
if (condition) doSomething();
```

### Comments
- ✅ JavaDoc required for public APIs
- ✅ Implementation comments explain WHY, not WHAT
- ✅ Keep comments updated with code changes
```java
/**
 * Processes the order and notifies relevant parties.
 * 
 * @param order The order to process
 * @return true if processing successful
 * @throws OrderProcessingException if validation fails
 */
public boolean processOrder(Order order) {
    // Validate before processing to prevent invalid state
    validateOrder(order);
    return true;
}
