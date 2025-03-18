# Java Code Review Guidelines

This repository contains comprehensive guidelines for Java code reviews, organized in multiple phases.

## Phase 1: Core Guidelines

These guidelines cover fundamental aspects of Java development:

1. **Naming and Style**
   - File: [naming_and_style.md](naming_and_style.md)
   - Covers naming conventions, code organization, and formatting

2. **Exception Handling**
   - File: [exception_handling.md](exception_handling.md)
   - Best practices for handling and propagating exceptions

3. **Javadoc Guidelines**
   - File: [javadoc-guidelines.md](javadoc-guidelines.md)
   - Comprehensive guide for writing effective documentation
   - Package, class, method, and field documentation
   - Best practices and examples

4. **Logging**
   - File: [logging.md](logging.md)
   - Guidelines for effective logging and monitoring

5. **Code Reusability**
   - File: [reusability.md](reusability.md)
   - SOLID principles and design patterns

6. **Static Code Analysis**
   - File: [static_code_analysis.md](static_code_analysis.md)
   - Code quality metrics and tools

7. **Validation**
   - File: [validation.md](validation.md)
   - Input validation and data integrity

## Phase 2: Advanced Guidelines

These guidelines cover more specialized aspects:

1. **Performance Optimization**
   - File: [performance.md](performance.md)
   - Best practices for optimizing Java applications

2. **Spring Framework**
   - File: [spring_best_practices.md](spring_best_practices.md)
   - Guidelines specific to Spring Framework development

3. **Memory Management**
   - File: [memory_leaks.md](memory_leaks.md)
   - Preventing and detecting memory leaks

## How to Use

1. Start with Phase 1 guidelines for fundamental code quality
2. Move to Phase 2 for optimization and framework-specific improvements
3. Apply guidelines progressively during code reviews
4. Use as reference during development

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

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
