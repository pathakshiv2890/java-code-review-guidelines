# Java Static Code Analysis Guidelines

## 1. Null Safety

- Avoid null pointer dereferences
- Use Optional for values that might be absent
```java
// Good - Null check
String name = getName();
if (name != null && name.equals("admin")) {
    // Process
}

// Good - Optional usage
Optional<User> user = findUser(id);
user.ifPresent(u -> sendEmail(u.getEmail()));

// Bad - Potential NPE
String name = getName();
if (name.equals("admin")) { // NPE if name is null
    // Process
}
```

## 2. Resource Management

- Handle or use return values from methods
- Close resources properly
```java
// Good - Handling return value
boolean removed = list.remove(item);
if (removed) {
    log.info("Item removed successfully");
}

// Bad - Ignored return value
list.remove(item); // Don't know if removal succeeded
```

## 3. Code Quality

- Remove unused variables, imports, and dead code
- Use equals() for object comparison, not ==
```java
// Good - Proper string comparison
if ("admin".equals(role)) {
    // Process
}

// Bad - Unused imports and variables
import java.util.Set; // Unused
public void process() {
    int count = 0; // Unused
    if (false) {
        log.info("Unreachable code");
    }
}
```

## 4. Concurrency

- Use thread-safe collections for shared data
- Minimize synchronization scope
```java
// Good - Thread-safe collection
private final List<String> items = Collections.synchronizedList(new ArrayList<>());
// OR
private final List<String> items = new CopyOnWriteArrayList<>();

// Bad - Unsafe concurrent access
private List<String> items = new ArrayList<>(); // Not thread-safe

// Bad - Overly broad synchronization
synchronized (this) { // Locks entire object
    // Only need to protect a small section
}
```

## 5. Complexity Management

- Limit nesting to 3 levels
- Keep methods under 30 lines
- Aim for cyclomatic complexity below 10
```java
// Good - Flat structure
public boolean isValidUser(User user) {
    if (user == null) return false;
    if (user.getId() <= 0) return false;
    if (user.getName() == null) return false;
    return true;
}

// Bad - Excessive nesting
public boolean isValidUser(User user) {
    if (user != null) {
        if (user.getId() > 0) {
            if (user.getName() != null) {
                return true;
            }
        }
    }
    return false;
}
```

## 6. SonarQube Guidelines

- Fix all "Blocker" and "Critical" issues
- Maintain code coverage above quality gate threshold
- Address code smells that affect maintainability

### Key SonarQube Rules
```java
// S1192: String literals should not be duplicated
// Bad
void process() {
    log.info("Processing started");
    // ... many lines later
    log.info("Processing started"); // Duplicated string
}

// S2095: Resources should be closed
// Good
try (Connection conn = dataSource.getConnection()) {
    // Use connection
}

// S1481: Unused local variables should be removed
// Bad
void process() {
    int count = 0; // Unused variable
    doSomething();
}

// S3776: Cognitive complexity should not be too high
// Bad - High cognitive complexity
boolean validate(String input) {
    if (input != null) {
        if (input.length() > 5) {
            for (int i = 0; i < input.length(); i++) {
                if (Character.isDigit(input.charAt(i))) {
                    return true;
                }
            }
        }
    }
    return false;
}
```

### SonarQube Quality Gates
- **Coverage**: Maintain > 80% test coverage
- **Duplication**: Keep duplicated lines < 3%
- **Maintainability**: Keep technical debt ratio < 5%
- **Reliability**: Zero bugs in code
- **Security**: Zero vulnerabilities in code

## 7. Common Fixes

- **Null Safety**: Use `Optional` or add null checks
- **Ignored Returns**: Capture and handle return values
- **Complexity**: Extract methods for deep nesting
```java
// Before - Complex nested logic
public boolean validate(User user) {
    if (user != null && user.getAddress() != null && 
        user.getAddress().getZipCode() != null &&
        user.getAddress().getZipCode().length() == 5) {
        return true;
    }
    return false;
}

// After - Extracted methods
public boolean validate(User user) {
    return hasValidZipCode(user);
}

private boolean hasValidZipCode(User user) {
    if (user == null) return false;
    Address address = user.getAddress();
    if (address == null) return false;
    String zipCode = address.getZipCode();
    return zipCode != null && zipCode.length() == 5;
}
``` 