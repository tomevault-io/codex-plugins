# General Principles
- Prioritize my project's local code structure and logic above all else.

## Writing Tests
- For newly created code or existing code functionality, make sure to write tests that cover the new functionality. This will help ensure that the new code is working correctly and will also help catch any potential issues or bugs early on.
- When writing tests, make sure to cover both positive and negative scenarios to ensure comprehensive test coverage. This will help identify any potential issues or edge cases that may arise in the code.
- Use descriptive test names that clearly indicate the purpose of the test and the expected outcome. This will make it easier for other developers to understand the intent of the test and the functionality being tested.
- Follow the existing test structure and conventions used in the project to maintain consistency and readability. This will help other developers easily navigate and understand the test suite, making it easier to maintain and update in the future.

## File & Directory Structure
- Each page lives in its own subdirectory under `src/renderer/src/pages/`. The directory name is the page name (e.g. `Explorer/`, `Query/`). All files that belong exclusively to that page — the component file, its CSS, sub-components, hooks, and utilities — go inside that directory.
- Components, hooks, utilities, or stylesheets that are shared between two or more pages go in `src/renderer/src/components/` (for UI components) or the appropriate shared directory, not inside a page directory.
- Shared page-level CSS (styles applied to all pages) lives at `src/renderer/src/pages/pages.css` and is imported with a relative path (`'../pages.css'`) from inside a page directory.

## Development Guidelines
- Performence is important, and the UI should feel smooth and responsive. When implementing new features or making changes to existing ones, consider the performance implications of your code and strive to optimize it for speed and efficiency. This may involve using techniques such as memoization, lazy loading, or optimizing rendering.
- Don not use Tailwind CSS for styling. Instead, use traditional CSS or a CSS-in-JS solution that is already being used in the project. This will help maintain consistency in the codebase and avoid introducing new dependencies or styles that may not fit well with the existing design.
- Follow the existing coding style and conventions used in the project to maintain consistency and readability. This will help other developers easily understand and contribute to the codebase.
- When implementing new features, make sure it has tests that cover the new functionality and run those tests to ensure that the new code is working correctly and does not introduce any new issues or bugs.
- Write clean, maintainable and reusable code that is easy to understand and modify in the future.
- Break down complex problems into smaller, manageable pieces to make it easier to implement and test the code.
- Use meaningful variable and function names that clearly indicate their purpose and functionality. This will help other developers understand the code more easily and quickly, making it easier to maintain and update in the future.
- Break the UI into reusable components that can be easily maintained and updated in the future. This will help ensure that the UI is consistent and easy to understand, while also making it easier to implement new features or make changes to existing ones without affecting the overall structure of the application.

## Documentation
- Maintain `docs/features.md` file that documents the features of the application in detail. This file should provide a comprehensive overview of the functionality and capabilities of the application, including any specific behaviors, limitations, or requirements.
- Document any new features or changes to existing features in the `docs/features.md` file as they are implemented. This will help ensure that the documentation remains up-to-date and accurate, providing users with the most current information about the application's capabilities.
- Use clear and concise language in the documentation to make it easy for users to understand the features and functionality of the application. This will help ensure that the documentation is accessible and useful to a wide range of users, regardless of their technical background or familiarity with the application.
- Manage release note file for a version. The file should be placed under `/docs` folder, and if it doesn't exist create it. The file name should be in the following format: `release_notes_vX.Y.Z.md` where "X.Y.Z" is the version number taken from `package.json` file. In the release notes maintain a list of features that are added so it can later be used as the application official release notes.

---
> Source: [developer82/Spiral](https://github.com/developer82/Spiral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-27 -->
