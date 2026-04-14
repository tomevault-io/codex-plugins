<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# ShareLink - File Sharing App Instructions

This is a Node.js Express application for instant file sharing with the following features:

## Key Features
- Drag & drop file uploads
- Custom short links with optional slugs
- PIN protection (4-digit)
- QR code generation for easy sharing
- Auto-expiry with TTL (time-to-live)
- Beautiful gradient UI with hover effects
- Sender/Receiver modes
- Multiple file bundling (auto-zip)
- Real-time progress indicators

## Tech Stack
- Node.js with Express server
- Multer for file uploads
- QRCode library for QR generation
- Archiver for ZIP creation
- In-memory storage (ephemeral)
- Responsive CSS with gradients and animations

## Color Scheme
- Primary: Cyan blue (#4ecdc4) to teal (#44a08d)
- Secondary: Pink (#ff6b6b) to red (#ee5a52)
- Gradients: Purple (#667eea) to violet (#764ba2)
- Accent: Pink gradients (#ffecd2 to #fcb69f)

## API Endpoints
- POST /api/upload - Upload multiple files
- GET /share/:slug - Download files (with PIN check)
- GET /api/share/:slug/info - Get share metadata
- GET /api/qr/:slug - Generate QR code
- GET /api/shares - List active shares
- DELETE /api/share/:slug - Delete share

## Design Principles
- Glassmorphism effects with backdrop-filter
- Smooth hover animations and transitions
- Glow effects on key elements
- Mobile-responsive design
- Accessibility-focused UI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ankit956021) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
