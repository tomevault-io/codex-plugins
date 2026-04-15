
# Instructions for Copilot
- Project is setup using nx workspace.
- packages are in the `packages` folder.
- The `packages` folder contains multiple packages.
- backend code is in the `packages/backend` folder. made with express
- shared code is in the `packages/shared` folder. made with typescript : TRPC ,mongoose-schema , zod 
- floder Structure for `packages/shared` is as follows:
    - shared
      - src 
        - lib
          - models   // mongoose schema and mongoose model
          - validation // zod validation
          - utils 
          - router // trpc router
          - types // typescript types
          - trpc.ts // trpc setup

- frontend code is in the `packages/frontend` folder. made with react
- frontend code is in the `packages/frontend/src` folder.
- Folder Structure for Frontend is as follows:
    - frontend
      - src
        - components // react components
        - pages // react pages
        - styles // css styles
        - utils // utils functions
          - trpc // trpc client
        - hooks // custom hooks
        - context // react context
        - app // app specific code
          - App.tsx // main app file
        - main.tsx // entry point file
        - router.tsx // react router file using create browser router

- using tailwindcss for styling
-configured colors and packages/frontend/src/styles.css

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shreyashbijlwanOP) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
