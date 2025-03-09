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

4. **Review with module leads**:
   - All review commits must be reviewed with respective module leads
   - Test cases must cover the modified sections
   - Include reviewer names in commit messages

5. **Production branch policy**:
   - Only merge and commit to main branch (production branches) after:
     - Functional tests are complete
     - Unit tests are complete
     - Module lead approval is obtained
   - Document test completion in merge requests

## Responsibility and Accountability

1. **Code ownership**:
   - Developers committing code are considered primary owners of that code
   - Reviewers share ownership and responsibility for the code they approve
   - Both parties are accountable for the quality and maintainability of the code

2. **Maintenance responsibilities**:
   - Code authors and reviewers are responsible for:
     - Debugging issues that arise in their code
     - Maintaining the code as requirements evolve
     - Addressing technical debt in their areas of ownership
     - Responding to production incidents related to their code

3. **Knowledge transfer**:
   - Code owners must document complex implementations
   - When transitioning projects, proper handover of code ownership must occur
   - Historical context and design decisions should be documented

4. **Continuous improvement**:
   - Code owners should proactively refactor and improve their code
   - Regular reviews of existing code should be scheduled
   - Technical debt should be addressed systematically

5. **AI-assisted code development**:
   - When using AI tools to generate or edit code:
     - Developers must thoroughly review and understand all AI-generated code
     - Developers are responsible for explaining AI-generated code to module leads
     - The use of AI tools does not reduce developer accountability for the code
   - Before committing AI-assisted code:
     - Verify correctness and adherence to project standards
     - Be prepared to explain the implementation details and reasoning
     - Ensure you can maintain the code without AI assistance in the future
