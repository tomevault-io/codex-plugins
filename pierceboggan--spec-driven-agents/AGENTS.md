<!-- These "custom instructions" are attached to each prompt from the user in chat.

Common information:
-Rules for interacting with me
-Team and organization context, like coding guidelines
-Project context, like the repository layout and technical architecture 

Learn more: https://code.visualstudio.com/docs/copilot/copilot-customization
-->

# Rules

### Rules
-Summarize the changes you made back to the user at a high level at end of response

## Code
-Add comments to the top of each file explaining what the file does at a very high level
-Add comments to each class, method, and member explaining what they do. Be precise, but brief

## User Interface
-Generate beautiful, user-friendly UI
-Keep nesting to a minimum
-Create composable UI widgets if appropriate
-UI must follow basic accessibility principles
-Generate tests in Playwright

## Tools
### edit_file
-Run git in the terminal to stage and commit changes with an AI-generated commit message when you are done using the edit_file tool

---
> Source: [pierceboggan/spec-driven-agents](https://github.com/pierceboggan/spec-driven-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-21 -->
