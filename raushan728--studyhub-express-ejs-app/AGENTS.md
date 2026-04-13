# Copilot Instructions for StudyHub

## Overview
StudyHub is a full-stack web application built with Express.js, EJS, MongoDB, and modern CSS animations. It provides a platform for students to learn, interact, and manage their educational resources. The project follows a modular architecture with clear separation of concerns.

## Architecture
- **Backend**: Express.js handles routing, middleware, and API logic.
  - Routes are organized by feature in the `routes/` directory (e.g., `auth.js`, `courses.js`, `notes.js`).
  - Middleware is stored in `middleware/` (e.g., `auth.js` for authentication).
  - Models are defined using Mongoose in the `models/` directory (e.g., `User.js`, `Course.js`, `Note.js`).
- **Frontend**: EJS templates in the `views/` directory provide dynamic HTML rendering.
  - Templates are organized by feature (e.g., `views/courses/`, `views/auth/`).
  - Shared components like navigation and footer are included using partials.
- **Static Assets**: CSS, JavaScript, and images are stored in the `public/` directory.
- **Database**: MongoDB is used for data storage, with Mongoose for schema definition and querying.

## Developer Workflows
### Running the Application
1. Install dependencies:
   ```cmd
   npm install
   ```
2. Set up environment variables in a `.env` file:
   ```env
   PORT=3000
   MONGO_URI=<your-mongodb-uri>
   SESSION_SECRET=<your-session-secret>
   ```
3. Start the application:
   ```cmd
   npm start
   ```

### Debugging
- Use `console.log` for debugging server-side code.
- Check the browser console for frontend issues.
- MongoDB queries can be debugged using Mongoose's `.exec()` method.

### Testing
- Manual testing is done by navigating through the application.
- Automated tests can be added using Mocha or Jest.

## Project-Specific Conventions
- **Authentication**:
  - Middleware (`isAuthenticated`, `isAdmin`) is used to protect routes.
  - Passwords are hashed using bcrypt.
- **Error Handling**:
  - Flash messages (`connect-flash`) are used for user feedback.
  - Errors are logged to the console.
- **Responsive Design**:
  - CSS media queries ensure the application works on mobile, tablet, and desktop.
- **File Uploads**:
  - Multer is used for handling file uploads (e.g., notes, course materials).

## Integration Points
- **External Dependencies**:
  - MongoDB for database storage.
  - bcrypt for password hashing.
  - Multer for file uploads.
- **Cross-Component Communication**:
  - Routes interact with models to fetch and update data.
  - Views use data passed from routes to render dynamic content.

## Examples
### Adding a New Feature
1. Create a new route file in `routes/` (e.g., `routes/quiz.js`).
2. Define the corresponding model in `models/` (e.g., `models/Quiz.js`).
3. Add EJS templates in `views/` (e.g., `views/quiz/`).
4. Update navigation links in `views/partials/navbar.ejs`.

### Common Patterns
- **Route Definition**:
  ```javascript
  router.get('/login', (req, res) => {
      res.render('auth/login', { title: 'Login - StudyHub' });
  });
  ```
- **Model Definition**:
  ```javascript
  const userSchema = new mongoose.Schema({
      name: String,
      email: { type: String, unique: true },
      password: String,
  });
  ```
- **EJS Template**:
  ```html
  <h1><%= title %></h1>
  ```

Feel free to suggest updates or clarify any sections!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raushan728) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
