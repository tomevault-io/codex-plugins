# TerzoTimeSheets Project Setup Instructions

## Project Overview
Next.js 14 application for managing employee timesheets for cafe-bars using Supabase.

## Setup Progress
- [x] Create copilot-instructions.md and scaffold Next.js project
- [x] Set up Supabase configuration and database schema
- [x] Implement authentication system
- [x] Create store and employee management
- [x] Build daily timesheet interface
- [x] Install dependencies and test
- [x] Add employee hiring/firing date system
- [x] Create reports page with weekly and monthly views
- [x] Implement user-store access control
- [x] Add daily comments feature for timesheets

## Key Features
- **Store Management**: Create and manage multiple cafe-bar locations
- **Employee Management**: Track employees with hiring and firing dates
- **Daily Timesheets**: Record clock-in/out times with 15-minute intervals
- **Daily Comments**: Add optional notes for each day below total hours
- **Reports**: Weekly and monthly hour summaries per employee
- **Access Control**: Users only see stores they're assigned to
- **Date Navigation**: Calendar picker for easy date selection

## Project Structure
- `/app` - Next.js App Router pages
- `/components` - React components
- `/lib` - Utility functions and Supabase client
- `/types` - TypeScript type definitions

## Next Steps
1. Set up a Supabase project
2. Run the SQL commands from DATABASE_SCHEMA.md to create tables and RLS policies
3. Create a .env.local file with your Supabase credentials
4. Create an admin user in Supabase Auth
5. Assign the admin user to stores (see USER_STORE_MANAGEMENT.md)
6. Run `npm run dev` to start the development server

## Important Files
- `DATABASE_SCHEMA.md` - Complete database schema and migration scripts
- `USER_STORE_MANAGEMENT.md` - Guide for assigning users to stores
- `DAILY_COMMENTS_FEATURE.md` - Documentation for daily comments feature
- `migration_daily_comments.sql` - Database migration for comments feature
- `.env.local.example` - Example environment variables

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cirrus3d) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
