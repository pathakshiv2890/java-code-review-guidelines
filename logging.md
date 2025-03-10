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

## 4. Structured Logging

- ✅ Use JSON format for logs in production
- ✅ Include standard fields in all log entries
- ✅ Enable easy log aggregation and searching

```java
// application.properties for JSON logging
spring.application.name=order-service
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} %p %c{1} [%X{traceId},%X{spanId}] %m%n
logging.level.root=INFO
logging.level.com.company=DEBUG
logging.logback.rollingpolicy.max-file-size=10MB
logging.logback.rollingpolicy.max-history=7

// Custom JSON appender in logback-spring.xml
<appender name="JSON_CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <includeMdc>true</includeMdc>
        <customFields>{"application":"${spring.application.name}"}</customFields>
    </encoder>
</appender>
```

## 5. Secure Logging

- ✅ Never log sensitive information (passwords, tokens, PII)
- ✅ Mask or hash sensitive data when needed
- ✅ Be careful with exception messages that might contain sensitive data

```java
// Good - Masking sensitive data
logger.info("Processing payment for card: {}", maskCardNumber(cardNumber));
logger.debug("User details: {}", maskUserDetails(user));

// Helper methods for masking
private String maskCardNumber(String cardNumber) {
    if (cardNumber == null || cardNumber.length() < 8) return "[INVALID CARD]";
    return "XXXX-XXXX-XXXX-" + cardNumber.substring(cardNumber.length() - 4);
}

// Bad - Logging sensitive information
logger.info("User password reset: {}", password); // Never log passwords
logger.debug("Authorization header: {}", authHeader); // Contains token
System.out.println("Credit card: " + cardNumber); // Extremely bad practice
```

## 6. Performance Considerations

- ✅ Use guard clauses to avoid expensive logging operations
- ✅ Consider async logging for high-throughput applications
- ✅ Monitor log volume and adjust levels accordingly

```java
// Good - Guard clause for expensive debug logging
if (logger.isDebugEnabled()) {
    logger.debug("Complex object state: {}", generateExpensiveDebugInfo(object));
}

// Good - Async logging configuration in logback-spring.xml
<appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="FILE" />
    <queueSize>512</queueSize>
    <discardingThreshold>0</discardingThreshold>
</appender>
```

## 7. Common Logging Mistakes

- ✅ Using System.out.println() instead of proper logging
- ✅ Logging at incorrect levels
- ✅ Catching exceptions without logging them
- ✅ Logging sensitive information

```java
// Bad practices to avoid
System.out.println("Debug info: " + debugInfo); // Use logger instead

try {
    riskyOperation();
} catch (Exception e) {
    // Silent catch - no logging
}

// Good alternative
try {
    riskyOperation();
} catch (Exception e) {
    logger.error("Failed during risky operation", e);
    // Or with ExceptionUtils
    logger.error("Stack trace: {}", ExceptionUtils.getStackTrace(e));
}
```

## 8. Avoid Unnecessary Logging

- ✅ Don't log for expected business scenarios
- ✅ Don't log exceptions that are part of normal flow
- ✅ Avoid duplicate logging across layers

