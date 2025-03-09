# Java Code Review Guidelines

This directory contains comprehensive guidelines for Java code reviews. Each aspect is detailed in its own file for better organization and focus.

## Core Guidelines

1. [Naming and Style](./naming_and_style.md) - Consistent naming conventions and code formatting
2. [Spring Best Practices](./spring_best_practices.md) - Spring Framework and Spring Boot best practices
3. [Exception Handling](./exception_handling.md) - Proper exception handling and error management
4. [Performance](./performance.md) - Performance optimization and efficiency
5. [Memory Leaks](./memory_leaks.md) - Preventing and detecting memory leaks
6. [Reusability](./reusability.md) - Code reuse and modular design
7. [Static Code Analysis](./static_code_analysis.md) - Automated code quality checks
8. [Validation](./validation.md) - Input validation and data integrity
9. [Logging](./logging.md) - Logging best practices and configuration

## How to Use

1. For each code review, start with the relevant guideline file based on the primary focus of the review
2. Use the guidelines provided in each file
3. Reference specific guidelines when providing feedback
4. Follow the examples provided in each guide

## Using Cursor for Code Review

Cursor provides powerful AI-assisted capabilities for code review and correction. Here's how to use Cursor with these guidelines:

### Code Review Workflow

1. **Open the codebase in Cursor**:
   - Open the project folder in Cursor
   - Navigate to the files you want to review

2. **Reference guidelines during review**:
   - Open the relevant guideline file in a split view
   - Use `/` command to search for specific guidelines

3. **AI-assisted review**:
   - Select code and press `Cmd+K` (Mac) or `Ctrl+K` (Windows/Linux)
   - Ask: "Does this code follow the [guideline] best practices?"
   - Example: "Does this code follow the exception handling best practices?"

4. **Identify issues**:
   - Use Cursor's AI to identify potential issues
   - Example prompt: "Find potential memory leaks in this code"
   - Compare findings against our memory_leaks.md guidelines

### Making Corrections

1. **AI-assisted fixes**:
   - Select problematic code
   - Press `Cmd+K` (Mac) or `Ctrl+K` (Windows/Linux)
   - Ask: "How can I fix this to follow [guideline]?"
   - Example: "How can I fix this to follow our logging guidelines?"

2. **Batch corrections**:
   - For similar issues across multiple files, use:
   - `Cmd+Shift+F` (Mac) or `Ctrl+Shift+F` (Windows/Linux) to find all occurrences
   - Ask Cursor AI to generate a fix that can be applied to all instances

3. **Validation**:
   - After making changes, ask Cursor AI to validate the changes
   - Example: "Does this code now follow our performance guidelines?"

### Team Review Process

1. **Share findings**:
   - Use Cursor's code snippets feature to share findings with team
   - Reference specific sections of the guidelines in comments

2. **Document patterns**:
   - Identify common issues across the codebase
   - Update guidelines if needed based on recurring patterns

3. **Learning loop**:
   - Use Cursor AI to explain why certain patterns are problematic
   - Ask: "Why is this logging approach not recommended?"
   - Use explanations to improve team understanding

## Quick Reference

- For general code style: See `naming_and_style.md`
- For API reviews: See `spring_best_practices.md`
- For error handling: See `exception_handling.md`
- For performance issues: See `performance.md`
- For memory management: See `memory_leaks.md`
- For code organization: See `reusability.md`
- For code quality: See `static_code_analysis.md`
- For input validation: See `validation.md`
- For logging practices: See `logging.md`

## Guidelines Focus

- **Naming and Style**: Consistent naming conventions, code formatting, and readability
- **Spring Best Practices**: Dependency injection, transaction management, REST API design
- **Exception Handling**: Custom exceptions, try-catch blocks, error responses
- **Performance**: Data structures, algorithms, resource optimization
- **Memory Leaks**: Resource management, reference handling, caching strategies
- **Reusability**: DRY principle, SOLID principles, modular design
- **Static Code Analysis**: Null safety, code quality, complexity management
- **Validation**: Input validation, boundary checks, defensive programming
- **Logging**: Log levels, structured logging, secure logging practices

## Contributing

To add or modify guidelines:
1. Create a new markdown file following the existing naming pattern
2. Update this README.md to include the new guide
3. Follow the template structure provided in `_template.md` 