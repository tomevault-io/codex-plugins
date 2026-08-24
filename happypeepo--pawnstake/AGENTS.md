# PawnStake - Full-Stack Chess Application

## Project Overview
A real-time chess application with React frontend, Node.js/Express backend, Socket.io for real-time communication, PostgreSQL with Prisma ORM, and Stockfish.js for move evaluation.

## Tech Stack
- **Frontend**: React.js, Tailwind CSS, react-chessboard, chess.js
- **Backend**: Node.js, Express, Socket.io
- **Database**: PostgreSQL with Prisma ORM
- **Engine**: Stockfish.js (WebAssembly)
- **Auth**: JWT-based authentication

## Project Structure
```
PawnStake/
├── client/                 # React frontend
│   ├── src/
│   │   ├── components/     # React components
│   │   ├── context/        # React context providers
│   │   ├── hooks/          # Custom hooks
│   │   ├── pages/          # Page components
│   │   ├── services/       # API services
│   │   └── utils/          # Utility functions
│   └── public/
├── server/                 # Node.js backend
│   ├── src/
│   │   ├── controllers/    # Route controllers
│   │   ├── middleware/     # Express middleware
│   │   ├── routes/         # API routes
│   │   ├── services/       # Business logic
│   │   ├── socket/         # Socket.io handlers
│   │   └── utils/          # Utility functions
│   └── prisma/             # Prisma schema and migrations
└── README.md
```

## Development Commands
- Frontend: `cd client && npm run dev`
- Backend: `cd server && npm run dev`
- Database: `cd server && npx prisma migrate dev`

## Environment Variables
See `.env.example` files in client and server directories.

---
> Source: [happypeepo/PawnStake](https://github.com/happypeepo/PawnStake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-21 -->
