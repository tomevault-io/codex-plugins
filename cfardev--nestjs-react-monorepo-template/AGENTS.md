This project follows the "Screaming Architecture" pattern.

# Screaming Architecture for React 

In page folder, create a folder for each module.

For example, if we have a page called "tasks", we will create a folder called "tasks" in the pages folder.

# Estructure of the pages folder

```
    /components
    /hooks
    /services
    /types
    /utils
    /pages
        /home
            /components
            /hooks
            /services
            /types
            /utils
            /index.tsx
        /tasks
            /components
              /task-list.tsx
              /task-item.tsx
              /task-form.tsx
              /task-detail.tsx
            /hooks
            /services
            /types
            /utils
            /index.tsx
        
```
The folders outside the pages folder are shared between all pages, and the folders inside the pages folder are specific to the page.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cfardev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/cfardev)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
