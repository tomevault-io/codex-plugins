# Copilot Instructions - TIN200 CRUD Application

## Project Overview
ASP.NET Core web application with SQL Server CRUD interface for the TIN200 database table.

## Setup Checklist

- [x] Scaffold the ASP.NET Core project
- [x] Create Entity Framework DbContext and models
- [x] Create CRUD page handlers and services
- [x] Create Razor Pages views for CRUD operations
- [x] Configure SQL Server connection string
- [ ] Create and apply database migrations
- [ ] Test and verify the application

Verification notes:
- Project publish build completed successfully on 2026-06-11.
- Migration scripts are present in Migrations/, but database apply state is not verified from this workspace.

## Key Features
- Create, Read, Update, Delete operations for TIN200 table
- Responsive web UI using Razor Pages
- Entity Framework Core for data access
- SQL Server database integration
- Form validation and error handling

## Database Schema
Table: TIN200
- Id (int, IDENTITY)
- CEO First Name (varchar 255)
- CEO Last Name (varchar 255)
- Email (varchar 255)
- External ID (varchar 50)
- Company Name (varchar 255)
- Company Description (varchar 255)
- FYE 2025 (decimal 18,0)
- FYE 2024 (decimal 18,0)
- FYE 2023 (decimal 18,0)
- TIN200 (varchar 50)

---
> Source: [NicholasJohnGriffiths/TIN200](https://github.com/NicholasJohnGriffiths/TIN200) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
