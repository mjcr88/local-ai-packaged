# Coding Standards for AI-Assisted Development (Tailored for Copilot)

This document provides coding standards and best practices to be followed when writing code, with specific guidance for working alongside an AI coding assistant like GitHub Copilot. These rules synthesize principles from Clean Code, Code Quality, Gitflow, Database Best Practices, and language/framework-specific guidelines (Python, Node.js/Express, Next.js/TypeScript/Tailwind).

**General Principles:**

1.  **Prioritize Clarity and Readability:** Code should be easy for humans to read and understand. Favor explicit over implicit.
2.  **Follow the Principle of Single Responsibility (SRP):** Each function, class, or module should have one, and only one, reason to change. Functions should do one thing and do it well.
3.  **Don't Repeat Yourself (DRY):** Avoid code duplication. Extract common logic into reusable functions, classes, or modules. Maintain a single source of truth for data and logic.
4.  **Verify Information:** Always assume generated code might contain subtle errors or misinterpretations. Review generated code carefully, especially for configuration, sensitive logic (auth, data handling), and integration points.
5.  **File-by-File Changes:** Apply changes incrementally, focusing on one file or a small set of related files at a time. This makes review and debugging easier.

**Naming Conventions:**

6.  **Use Meaningful Names:** Names for variables, functions, classes, modules, and files should be descriptive and clearly indicate their purpose, content, or behavior. Avoid generic names (`data`, `item`, `handler`) where a more specific name is possible (`userData`, `currentItem`, `requestHandler`).
7.  **Follow Language/Framework Conventions:**
    * **Python:** `snake_case` for functions and variables, `PascalCase` for classes, `UPPER_CASE` for constants.
    * **JavaScript/TypeScript:** `camelCase` for variables and functions, `PascalCase` for classes and components.
    * **Files/Directories:** Use lowercase with dashes (kebab-case) for directory and non-code file names (e.g., `user-authentication`, `data-processing`). Code files should follow language conventions (e.g., `my_module.py`, `myComponent.tsx`).
8.  **Use Constants:** Replace "magic numbers" or hardcoded strings with named constants. Use descriptive constant names (e.g., `DEFAULT_TIMEOUT_SECONDS`, `API_BASE_URL`). Place constants at the top of the file or in a dedicated constants file.

**Code Structure & Organization:**

9.  **Keep Related Code Together:** Organize code logically into directories and modules. Group related functions, classes, or components together.
10. **Use Standard Project Layouts:** Follow established conventions like `src/` layouts (Python, Node.js) and the App Router structure (Next.js).
11. **Modularize Configuration:** Keep configuration separate from code, ideally using environment variables loaded from `.env` files (and managed outside version control).

**Comments & Documentation:**

12. **Explain *Why*, Not *What*:** Use comments to explain the *reasoning* behind a piece of code, complex logic, potential side effects, or workarounds, rather than simply restating what the code does (which should be clear from the code itself).
13. **Document APIs and Interfaces:** Clearly document public functions, classes, and APIs, including parameters, return values, and behavior. Use docstrings (Python) or JSDoc (JavaScript/TypeScript).
14. **Keep READMEs Updated:** Maintain a clear `README.md` at the root and within service directories explaining setup, purpose, and usage.

**Git & Version Control:**

15. **Follow Gitflow Workflow:** Adhere to the branch structure (main, develop, feature, release, hotfix) and branching rules defined in your `gitflow.pdf`.
16. **Make Small, Focused Commits:** Each commit should represent a single logical change. Write clear, concise commit messages using the `type(scope): description` format (feat, fix, docs, style, refactor, test, chore).
17. **Use Meaningful Branch Names:** Follow conventions like `feature/[issue-id]-descriptive-name`.
18. **Use Pull Requests:** All changes to main and develop branches must go through pull requests with required reviews.

**Error Handling & Robustness:**

19. **Implement Proper Error Handling:** Use try-except blocks (Python), async/await with error handling (Node.js), and error boundaries (React/Next.js). Don't just catch errors; log them and provide meaningful responses to the user or calling service.
20. **Validate Input:** Always validate user input at the boundaries of your system (e.g., API endpoints, form submissions) to prevent security vulnerabilities and unexpected behavior.
21. **Handle Edge Cases:** Explicitly consider and handle potential edge cases and error conditions during development.

**Database Interaction (Relevant to DB code or ORM usage):**

22. **Use Proper Schema Design:** Follow database normalization principles.
23. **Use Migrations:** Manage database schema changes using migration tools (e.g., Alembic for SQLAlchemy, Prisma Migrate).
24. **Implement Proper Indexing:** Ensure database queries are performant by adding appropriate indexes.
25. **Use ORMs/ODMs:** Utilize object-relational mappers (SQLAlchemy, Prisma) or object-document mappers to interact with databases, following their best practices for queries, relationships, and transactions.
26. **Handle Sensitive Data Securely:** Do not log sensitive data. Use proper encryption for data at rest and in transit.

**AI Assistant Specific Guidance (For Copilot):**

27. **Be Explicit in Prompts:** Clearly state your intent, the context (file, function, goal), and any constraints (language, framework, desired pattern) when asking for code generation or modification.
28. **Review Generated Code Thoroughly:** Do not blindly accept generated code. Read it line by line, understand what it does, check against the standards in this document, and verify its correctness and efficiency.
29. **Iterate and Refine:** If the initial generated code isn't quite right, provide specific feedback and constraints to guide the assistant towards the desired outcome.
30. **Apply Standards Manually if Needed:** If the assistant fails to follow a specific standard after guidance, correct the code manually and potentially adjust your prompting strategy.
31. **Focus on Implementation, Not Speculation:** Expect the assistant to provide code for the task requested, based on the provided context and standards. Do not expect it to invent requirements or make design decisions beyond the immediate coding task.
32. **Use `coding_standards.md` as Context:** Ensure this document is available in the workspace context when interacting with Copilot in VS Code, so it can learn from your preferred rules.

---

This document provides a solid base. As you work, you might discover more specific patterns or rules that are important for your project. You should update this document as needed to reflect your evolving standards.
