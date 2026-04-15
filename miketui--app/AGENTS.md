# Haus of Basquiat - Project Documentation

## 1. Project Overview

This project is a members-only social platform for the ballroom and voguing community. It features role-based access control, real-time messaging, media galleries, event management, and subscription payments.

## 2. Directory Structure

```
/App
├── backend/
│   ├── node_modules/
│   ├── index.js
│   ├── package.json
│   └── package-lock.json
├── database/
│   ├── schema.sql
│   └── setup-storage.sql
├── node_modules/
├── src/
│   ├── app/
│   │   ├── admin/
│   │   ├── chats/
│   │   ├── feed/
│   │   ├── gallery/
│   │   ├── houses/
│   │   ├── library/
│   │   ├── notifications/
│   │   ├── profile/
│   │   └── settings/
│   ├── assets/
│   ├── components/
│   ├── context/
│   ├── lib/
│   ├── pages/
│   ├── App.jsx
│   ├── index.css
│   └── main.jsx
├── .env.example
├── .eslintrc.json
├── CLAUDE.md
├── index.html
├── package.json
├── README.md
└── vite.config.js
```

## 3. Tech Stack

*   **Frontend:** React (with Vite), Tailwind CSS
*   **Backend:** Node.js, Express.js
*   **Database:** PostgreSQL (with Supabase)
*   **Authentication:** Supabase Auth (Magic Links)
*   **Real-time:** Socket.IO (for future implementation)
*   **Payments:** Stripe

## 4. Setup and Installation

### 4.1. Database Setup

1.  Create a new project on [Supabase](https://supabase.com/).
2.  Navigate to the **SQL Editor** in your Supabase project.
3.  Execute the contents of `database/schema.sql`.
4.  Execute the contents of `database/setup-storage.sql`.

### 4.2. Backend Setup

1.  Navigate to the `backend` directory:
    ```bash
    cd backend
    ```
2.  Install the dependencies:
    ```bash
    npm install
    ```
3.  Create a `.env` file by copying the `.env.example` file from the root directory.
4.  Fill in the environment variables in the `.env` file with your Supabase project URL, service key, Stripe secret key, and other required credentials.
5.  Start the backend server:
    ```bash
    npm start
    ```

### 4.3. Frontend Setup

1.  Navigate to the root `App` directory.
2.  Install the dependencies:
    ```bash
    npm install
    ```
3.  Start the frontend development server:
    ```bash
    npm run dev
    ```

## 5. Available Scripts

### 5.1. Backend (`/backend`)

*   `npm start`: Starts the server in production mode.
*   `npm run dev`: Starts the server in development mode with hot-reloading.
*   `npm test`: Runs the test suite.

### 5.2. Frontend (`/`)

*   `npm run dev`: Starts the development server.
*   `npm run build`: Builds the application for production.
*   `npm run preview`: Previews the production build.
*   `npm run lint`: Lints the source code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miketui)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/miketui)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
