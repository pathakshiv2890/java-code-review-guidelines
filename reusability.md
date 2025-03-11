# Java Code Reusability Guidelines

## 1. DRY Principle (Don't Repeat Yourself)

- Extract repeated logic into reusable methods or utility classes
- Use template methods for common workflows with variable steps
```java
// Good - Reusable utility method
public class StringUtils {
    public static String formatName(String first, String last) {
        return first + " " + last;
    }
}

// Bad - Duplicated logic
public String getUserName(User user) {
    return user.getFirstName() + " " + user.getLastName();
}
public String getEmployeeName(Employee employee) {
    return employee.getFirstName() + " " + employee.getLastName(); // Duplicated
}
```

## 2. SOLID Principles

- **Single Responsibility**: One class, one purpose
- **Open/Closed**: Extend functionality without modifying existing code
- **Dependency Inversion**: Depend on abstractions, not implementations
```java
// Good - Single responsibility & abstractions
public interface PaymentProcessor {
    void processPayment(Payment payment);
}

public class PaymentService {
    private final PaymentProcessor processor;
    
    public PaymentService(PaymentProcessor processor) {
        this.processor = processor;
    }
    
    public void pay(Payment payment) {
        processor.processPayment(payment);
    }
}
```

## 3. Modular Design

- Keep methods under 30 lines and focused on one task
- Favor composition over inheritance
- Build complex behavior from simple, reusable components
```java
// Good - Composition
public class ReportGenerator {
    private final DataFetcher dataFetcher;
    private final ReportFormatter formatter;
    
    // Components injected, not inherited
    public ReportGenerator(DataFetcher dataFetcher, ReportFormatter formatter) {
        this.dataFetcher = dataFetcher;
        this.formatter = formatter;
    }
}
```

## 4. Design Patterns for Reusability

- **Factory Pattern**: Hide complex object creation
- **Strategy Pattern**: Make algorithms interchangeable
- **Builder Pattern**: Construct complex objects step by step
```java
// Factory pattern example
public class ConnectionFactory {
    public static Connection createConnection(String type) {
        return switch (type) {
            case "mysql" -> new MySQLConnection();
            case "postgres" -> new PostgreSQLConnection();
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}
``` 