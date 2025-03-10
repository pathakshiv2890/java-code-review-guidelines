# Java Logging Guidelines

## 1. Log Levels

- ✅ Use appropriate log levels based on message purpose
- ✅ Configure different log levels for different environments

| Level | Purpose | Example Usage |
|-------|---------|---------------|
| ERROR | Runtime errors, exceptions that impact functionality | `Failed to process payment`, `Database connection failed` |
| WARN | Potential issues, degraded functionality | `Retry attempt 3 of 5`, `Using fallback service` |
| INFO | Key business events, application lifecycle | `Order #1234 created`, `Application started` |
| DEBUG | Detailed information for troubleshooting | `Processing order with 5 items`, `Query executed in 150ms` |
| TRACE | Very detailed diagnostic information | `Entering method with params: x=5, y=10`, `HTTP headers received` |

```java
// Good - Appropriate log levels
logger.error("Failed to process payment: {}", paymentId, exception);
logger.warn("Retry attempt {} of {}", attempt, maxAttempts);
logger.info("Order {} created successfully", orderId);
logger.debug("Query executed in {}ms", executionTime);
logger.trace("Method entry: findUserById({})", userId);

// Bad - Incorrect log levels
logger.error("Application started"); // Should be INFO
logger.info("SQL query execution plan: {}", plan); // Should be DEBUG

// Bad - Using System.out instead of logger
System.out.println("User logged in: " + username); // Use logger instead
```

## 2. Log Message Content

- ✅ Include contextual information (IDs, timestamps, user info)
- ✅ Use parameterized logging to avoid string concatenation
- ✅ Include stack traces for exceptions if generic exception is caught
- ✅ Use ExceptionUtils for better stack trace handling

```java
// Good - Contextual information
logger.info("User {} performed {} on resource {}", 
    userId, action, resourceId);

// Good - Exception with stack trace using ExceptionUtils
try {
    processOrder(order);
} catch (OrderProcessingException e) {
    logger.error("Failed to process order {}: {}", 
        order.getId(), e.getMessage(), e);
    
    //  Apache Commons ExceptionUtils to be used instead of e.printStackTrace()
    logger.error("Failed to process order {}: {}", 
        order.getId(), ExceptionUtils.getStackTrace(e));
}

// Bad - String concatenation
logger.info("Processing order " + order.getId() + " with " + 
    order.getItems().size() + " items"); // Performance issue

// Bad - Missing context
logger.error("Operation failed"); // No details about what failed
```

## 3. Spring Boot Logging

- ✅ Use Spring's built-in logging framework
- ✅ Configure trace and span IDs for distributed tracing
- ✅ Use Mapped Diagnostic Context (MDC) for request context
- ✅ Never use System.out.println() for logging

```java
// application.properties
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{traceId},%X{spanId}] %-5level %logger{36} - %msg%n
logging.level.root=INFO
logging.level.com.company=DEBUG
logging.level.org.springframework=WARN
logging.file.name=application.log
logging.file.max-size=10MB
logging.file.max-history=7

```

## 4. Secure Logging

- ✅ Never log sensitive information (passwords, tokens, PII)
- ✅ Instead if logging sensitive information, log the entity id and the field that contains the sensitive data
- ✅ Be careful with exception messages that might contain sensitive data

```java
// Good - Masking sensitive data
logger.info("Processing payment for card of user {}", userName);
// Bad - Logging sensitive information
logger.debug("User details: {}", maskUserDetails(user));


// Bad - Logging sensitive information
logger.info("User password reset: {}", password); // Never log passwords
logger.debug("Authorization header: {}", authHeader); // Contains token
System.out.println("Credit card: " + cardNumber); // Extremely bad practice
```

## 5. Avoid Unnecessary Logging

- ✅ Don't log for expected business scenarios
- ✅ Don't log exceptions that are part of normal flow
- ✅ Avoid duplicate logging across layers

