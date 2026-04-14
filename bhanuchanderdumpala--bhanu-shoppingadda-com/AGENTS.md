# Copilot Instructions for Bhanu_shoppingadda.com

## Project Overview
- **Type:** Node.js/Express web application for an e-commerce platform
- **Main entry:** `server.js` (uses `bin/www` for startup)
- **Frontend:** Static HTML/CSS/JS in `public/` (with product, order, and user flows)
- **Backend:** Route handlers in `routes/`, business logic and utilities in `routes/common/`
- **Views:** Jade templates in `views/` (for server-rendered pages)

## Key Architectural Patterns
- **Routes:** Each file in `routes/` handles a specific domain (e.g., `addNewProducDetails.js`, `orderslist.js`).
- **Data:** Product and order data are stored as JSON files in `public/data/` (no database by default).
- **Session/User:** User authentication and session logic in `routes/validateUserCredentials.js`, `routes/destroySession.js`.
- **Uploads:** File uploads handled in `routes/uploadFiles.js`.
- **Frontend JS:** Page-specific logic in `public/javascripts/` (e.g., `addNewProduct.js`, `orderslist.js`).
- **Static Assets:** Images in `public/images/`, product images in `public/images/products/`.

## Developer Workflows
- **Start server:** `node ./bin/www` (or use `npm start` if defined in `package.json`)
- **No formal test suite** (add tests if needed)
- **Debugging:** Use `console.log` in route handlers or frontend JS; inspect JSON data files for state
- **Add new product:** Update `public/data/newproduct.json` and images in `public/images/products/`

## Project-Specific Conventions
- **Route naming:** Route files are named for their function (e.g., `addNewProducDetails.js` for product creation)
- **Data persistence:** All product/order data is file-based (JSON), not a database
- **Frontend-backend communication:** Uses AJAX (see `public/javascripts/` for API calls)
- **No ES6 modules:** Uses CommonJS (`require`) throughout
- **Jade views:** Use `.jade` templates for server-rendered pages; static HTML for most frontend

## Integration Points & Dependencies
- **Bootstrap & Icons:** Included in `public/lib/bootstrap/` and `public/lib/bootstrap-icons/`
- **jQuery:** Used for DOM manipulation and AJAX (see `public/lib/jquery.js`)
- **Axios:** Used for HTTP requests (see `public/lib/axios.js`)
- **Custom libraries:** Captcha (`public/lib/capthcalib.js`), rating stars (`public/lib/ratingStars_lib/`)

## Examples
- **Add product flow:**
  - Frontend: `public/addNewProduct.html` + `public/javascripts/addNewProduct.js`
  - Backend: `routes/addNewProducDetails.js` (handles POST)
  - Data: `public/data/newproduct.json`, images in `public/images/products/`
- **Order list:**
  - Frontend: `public/orderslist.html` + `public/javascripts/orderslist.js`
  - Backend: `routes/orderslist.js`
  - Data: `public/data/`

## Recommendations for AI Agents
- When adding new features, follow the file-based data pattern (update JSON, not a DB)
- Place new route handlers in `routes/` and link from `server.js`
- For new frontend features, add HTML to `public/`, JS to `public/javascripts/`, and CSS to `public/css/`
- Use existing patterns for AJAX and data updates
- Reference `README.md` for high-level project info (currently minimal)

---
For questions about project structure or conventions, review `routes/`, `public/`, and `server.js` for examples.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bhanuchanderdumpala) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
