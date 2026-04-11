
# INSTRUCTIONS

- .NET 10
- Vue (latest version)
- MS SQL DB
- Docker

## General Context

I'd like to create a habit tracking web application, so I can log information relevant to the habit each day. To begin, I'd like to focus on a specific habit to form, eating better, and reducing bad food and drink intake. The application won't be limited to this, other ideas may simply be such as going to sleep earlier, exercising more, reducing screen time, or reading each day. These can be different sections of the application. 

But for now let's start with the food tracking one. They say it takes an average of 66 days to form or break a habit, so that is the baseline. With the better eating habit, I'd like the following:

Each day, be able to log what I eat and drink. This would be breakfast, morning tea, lunch, dinner and supper. These aren't required to be logged for, for example I may skip breakfast or morning tea entirely.

When a user first signs up, they will be asked what habit they'd like to create. For now the only option will be the eating one of course.

## Coding

The backend, frontend and database will be in a docker container. When building the application, the docker container will be run locally on one my servers that is always running. It will need to have the basic ports exposed for the application to function.

The idea will be I can pull this repository down on another machine, and run a docker compose to build, pull relevant images and run the application. The database will be in a separate container, and the backend and frontend will be in another container together.

Please also take into account security best practices. For example, the backend should be protected against common vulnerabilities such as SQL injection, cross-site scripting (XSS), and cross-site request forgery (CSRF). The frontend should also be protected against XSS and CSRF attacks.

### Backend
The backend will be a .NET 10 ASP.NET REST API web application. Users must create an account to be able to start logging for a habit. In terms of account handling, use the built in .NET Identity system. The user will be able to log in and out, and the application will use JWT tokens for authentication.

For the data layer, use Entity Framework core for CRUD operations. To initially create the database, create SQL scripts that can be run to create the database and tables. The database will be MS SQL.

### Frontend
The frontend will be a Vue application. It will have a login page, and then the main page will be the habit tracking page. The habit tracking page will have a form to log the food and drink intake for the day, and a section to view the history of the logged information. If the user wishes to see history, they can see their historic data.

The styling for the frontend will be done using Material Design. The application will be responsive and work on both desktop and mobile devices. With respect to mobile, the application will be a PWA (Progressive Web App) so it can be installed on mobile devices and work offline. When the device goes online, the data will be synced with the backend. 

The application will also have push notifications to remind the user to log their habit each day.

### Testing

Backend will have basic unit tests covering the respective controllers, services  and data access layers. 

The frontend will have basic unit tests covering the components and services. For automation, playwright will be used to create end to end tests for the application.

## Other info

If there are other questions which need addressing in the plan phase, or other parts in time, feel more than free to ask me. When I answer a question, update this file with the answer so that it can be referred to later on.

## Answered Questions

| Topic | Decision |
|---|---|
| Ports | App container on **80** (ASP.NET serves both API and Vue SPA static files), DB on **1433** |
| Meal detail | Free-text description **+** a Healthy / Unhealthy rating per meal entry |
| Offline conflict | **Local device data wins** — client sends full day state on sync, overwrites server |
| Push notifications | **User-configurable** reminder time (stored per user, sent via VAPID) |
| Habit duration | Configurable input on onboarding UI, **default 66 days** |
| Auth | **Email/password only** via .NET Identity + JWT (no OAuth) |
| Containers | **2 containers**: `db` (SQL Server 2022) and `app` (multi-stage: Node builds Vue → dotnet publishes API → runtime image serves both via `UseStaticFiles` + SPA fallback) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shaun-hutch)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/shaun-hutch)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
