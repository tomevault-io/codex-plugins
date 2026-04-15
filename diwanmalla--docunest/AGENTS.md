# DocuNest - Copilot Instructions

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview

DocuNest is a full-stack note-saving application built with Next.js 15 that allows users to upload, manage, and view PDF/DOCX files online.

## Tech Stack

- **Frontend**: Next.js 15 (App Router), React, TypeScript
- **Styling**: Tailwind CSS, Shadcn UI
- **Database**: Prisma with PostgreSQL
- **Authentication**: Clerk
- **File Storage**: Cloudinary
- **PDF Viewing**: react-pdf
- **File Upload**: React Dropzone

## Key Features

- User authentication and authorization
- File upload (PDF/DOCX) with cloud storage
- Online PDF viewer
- File management dashboard
- Secure file access with Cloudinary URLs

## Code Style Guidelines

- Use TypeScript for all new files
- Follow Next.js 15 App Router conventions
- Use Tailwind CSS for styling
- Implement proper error handling and loading states
- Use Prisma for database operations
- Follow RESTful API design for API routes
- Implement proper security measures for file access

## File Structure

- `/src/app` - Next.js App Router pages and API routes
- `/src/components` - Reusable React components
- `/src/lib` - Utility functions and configurations
- `/prisma` - Database schema and migrations
- `/public` - Static assets

## Environment Variables

Ensure proper environment variables are set for:

- Database connection (DATABASE_URL)
- Clerk authentication (NEXT*PUBLIC_CLERK*\*)
- Cloudinary configuration (CLOUDINARY_CLOUD_NAME, CLOUDINARY_API_KEY, CLOUDINARY_API_SECRET)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DiwanMalla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
