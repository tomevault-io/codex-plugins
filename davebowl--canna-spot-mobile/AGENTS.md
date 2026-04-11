# CannaSpot v3.6 – AI Agent Guide

**Architecture**: Monolithic Flask app with SQLAlchemy ORM. Models in `models.py`, routes in `app.py`, templates use Jinja2 inheritance from `base.html`.

## Quick Start
```bash
python app.py  # Starts dev server on :5000
```
Visit `/install` first run to configure DB (SQLite default, MySQL via `DATABASE_URL` env) and create admin account.

## Data Model (`models.py`)
- **User**: `username`, `email`, `password_hash` (SHA-256), `is_admin`
- **Server**: grow server with `slug` (via `safe_slug()`), has many `Channel`
- **Video**: `filename` (relative path `/uploads/videos/`), `thumbnail`, `uploader_id`, `created_at`
- **Message**: chat messages in channels
- Models imported into `app.py` via `from models import db, User, Server, ...`

## Template & Frontend Patterns
- `base.html`: Server icons leftbar + CannaSpot nav, header with Create dropdown, footer sponsors
- CSS in `static/css/`: `style.css` (main), `channel-nav.css` (leftbar), `discord.css` (server channels)
- Mobile: hamburger toggle (ID navToggle) shows off-canvas leftbar at <980px; uses nav-open body class
- Jinja filters: `@app.template_filter("date")` formats `datetime`/`date` objects (default `"%b %d, %Y"`)

## URL Patterns & Route Variables
- `/watch/<int:vid>`: pass `video=`, `related=`, `comments=[]` to template (`watch.html` expects these names)
- `/server/<slug>` → `/c/<slug>/<int:cid>` for channels
- All video links use `url_for('watch', vid=v.id)` (NOT `video_id=`)

## Key Conventions
- **Slugs**: `safe_slug(name)` strips non-alphanum, lowercases, hyphenates
- **Passwords**: SHA-256 via `hash_pw(pw)` in `models.py`
- **File uploads**: `secure_filename()`, store in `uploads/{videos,thumbnails,avatars}/`, serve via `/uploads/<path:filename>` route
- **Sessions**: `session["uid"]` for auth; `current_user()` helper returns `User` or `None`

## Adding Features
1. **New route**: Add to `app.py`, use `@app.route()`, call `render_template()` with exact var names template expects
2. **New model**: Add class in `models.py`, run `db.create_all()` or migrate
3. **New page**: Extend `base.html`, reference static assets via `url_for('static', filename='...')`
4. **Mobile**: Add `@media (max-width:980px)` in `style.css` for responsive tweaks

## Current State
- Mobile nav implemented (hamburger + off-canvas leftbar)
- Video player responsive (aspect-ratio 16:9, centered in `.wrap`)
- No comments backend yet (route passes `comments=[]`)
- No view counter on `Video` model (template shows "Uploaded <date>" only)

## API Routes (JSON endpoints)
- **POST `/api/like/<int:vid>`**: Like a video (creates `VideoLike` record)
- **POST `/api/watch-later/<int:vid>`**: Add to watch later (creates `WatchLater` record)
- **POST `/api/subscribe/<int:uid>`**: Subscribe to uploader (creates `Subscription` record)
- All require `current_user()` session; return `{"success": True}` or `{"error": "Not logged in"}`


## Core Components

### Data Models (`app.py`)
- `User`: Manages user accounts, profiles, and admin status
- `Server`: Represents grow servers with channels
- `Channel`: Chat channels within servers
- `Video`: Handles video uploads with thumbnails
- `Message`: Chat messages in channels
- `Sponsor`: Manages sponsor information
- `Activity`: Tracks system events

### File Structure
```
/uploads/
  /avatars/     # User profile pictures
  /thumbnails/  # Video thumbnails
  /videos/      # Uploaded video content
/static/        # Static assets
/templates/     # Jinja2 HTML templates
```

## Key Development Workflows

### Initial Setup
1. Run the application: `python app.py`
2. Access installation page at `/install` to:
   - Configure database (SQLite or MySQL)
   - Set up admin account
   - Initialize basic server structure

### Database Configuration
- Default: SQLite (`cannaspot.db` in project root)
- MySQL option available via environment variables:
  - `DATABASE_URL`: Connection string
  - `SECRET_KEY`: App secret key

### Media Handling
- Videos stored in `/uploads/videos/`
- File size limit: 512MB
- Thumbnails automatically stored in `/uploads/thumbnails/`

## Project-Specific Patterns

### URL Structure
- Server URLs: `/server/<slug>`
- Channel URLs: `/c/<slug>/<channel_id>`
- Video URLs: `/watch/<video_id>`

### Security Patterns
- Password hashing: SHA-256 (see `hash_pw()` function)
- Session-based authentication using Flask sessions
- Admin access controlled via `is_admin` flag

### Naming Conventions
- Server slugs: Generated using `safe_slug()` - lowercase, hyphenated
- File paths: Always use `os.path.join()` for cross-platform compatibility

## Environment Variables
- `DATABASE_URL`: Database connection string
- `SECRET_KEY`: Flask session secret key

## Integration Points
- MySQL database support via PyMySQL
- Static file serving through Flask's `send_from_directory`
- Template inheritance using Jinja2 base templates

## Core Components

### Data Models (`models.py`)
- `User`: Manages user accounts, profiles, and admin status
- `Server`: Represents grow servers with channels
- `Channel`: Chat channels within servers
- `Video`: Handles video uploads with thumbnails
- `Message`: Chat messages in channels
- `Sponsor`: Manages sponsor information
- `Activity`: Tracks system events

### File Structure
```
/uploads/
  /avatars/     # User profile pictures
  /thumbnails/  # Video thumbnails
  /videos/      # Uploaded video content
/static/        # Static assets
/templates/     # Jinja2 HTML templates
```

## Key Development Workflows

### Initial Setup
1. Run the application: `python app.py`
2. Access installation page at `/install` to:
   - Configure database (SQLite or MySQL)
   - Set up admin account
   - Initialize basic server structure

### Database Configuration
- Default: SQLite (`cannaspot.db` in project root)
- MySQL option available via environment variables:
  - `DATABASE_URL`: Connection string
  - `SECRET_KEY`: App secret key

### Media Handling
- Videos stored in `/uploads/videos/`
- File size limit: 512MB
- Thumbnails automatically stored in `/uploads/thumbnails/`

## Project-Specific Patterns

### URL Structure
- Server URLs: `/server/<slug>`
- Channel URLs: `/c/<slug>/<channel_id>`
- Video URLs: `/watch/<video_id>`

### Security Patterns
- Password hashing: SHA-256 (see `hash_pw()` function)
- Session-based authentication using Flask sessions
- Admin access controlled via `is_admin` flag

### Naming Conventions
- Server slugs: Generated using `safe_slug()` - lowercase, hyphenated
- File paths: Always use `os.path.join()` for cross-platform compatibility

## Environment Variables
- `DATABASE_URL`: Database connection string
- `SECRET_KEY`: Flask session secret key

## Integration Points
- MySQL database support via PyMySQL
- Static file serving through Flask's `send_from_directory`
- Template inheritance using Jinja2 base templates

## Deployment Guide

### Prerequisites
- Python 3.6+
- MySQL (if not using SQLite)
- DirectAdmin control panel
- Apache with Python support
- WSGI server (e.g., Gunicorn)

### DirectAdmin Deployment Steps
1. Create a Python application in DirectAdmin:
   - Log into DirectAdmin
   - Go to "Custom Application Versions"
   - Add Python 3.6+ if not already available

2. Configure Python Environment:
   ```bash
   cd ~/domains/your-domain.com/public_html
   python3 -m venv venv
   source venv/bin/activate
   ```

3. Upload Application:
   - Use DirectAdmin File Manager or FTP to upload all files
   - Ensure proper permissions:
   ```bash
   chmod -R 755 ~/domains/your-domain.com/public_html
   chmod -R 777 uploads/
   ```

4. Create custom handlers in DirectAdmin:
   - Go to "Custom HTTPD Configurations"
   - Add the following to your domain's configuration:

   ```apache
   # Python WSGI Configuration
   WSGIDaemonProcess cannaspot python-path=/home/username/domains/your-domain.com/public_html/venv/lib/python3.x/site-packages
   WSGIProcessGroup cannaspot
   WSGIScriptAlias / /home/username/domains/your-domain.com/public_html/app.wsgi

   # Static and Upload Directories
   Alias /static /home/username/domains/your-domain.com/public_html/static
   Alias /uploads /home/username/domains/your-domain.com/public_html/uploads

   <Directory /home/username/domains/your-domain.com/public_html>
       Require all granted
       WSGIScriptReloading On
   </Directory>

   <Directory /home/username/domains/your-domain.com/public_html/uploads>
       Require all granted
       Options -Indexes
       <FilesMatch "\.(jpg|jpeg|png|gif|mp4|webm)$">
           Require all granted
       </FilesMatch>
   </Directory>
   ```

5. Create WSGI file:
   Create `app.wsgi` in your domain root:
   ```python
   import sys
   import os

   # Add application directory to path
   sys.path.insert(0, '/home/username/domains/your-domain.com/public_html')

   # Activate virtual environment
   activate_this = '/home/username/domains/your-domain.com/public_html/venv/bin/activate_this.py'
   with open(activate_this) as file_:
       exec(file_.read(), dict(__file__=activate_this))

   # Import and create application
   from app import app as application
   ```

### Regular Deployment Steps
1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   pip install gunicorn
   ```

2. Configure environment variables:
   - Create `.env` file in project root
   - Set `SECRET_KEY` for production
   - Set `DATABASE_URL` if using MySQL

3. Set up Apache web server:
   ```apache
   # Enable required modules
   LoadModule proxy_module modules/mod_proxy.so
   LoadModule proxy_http_module modules/mod_proxy_http.so
   LoadModule headers_module modules/mod_headers.so
   LoadModule rewrite_module modules/mod_rewrite.so

   <VirtualHost *:80>
       ServerName your-domain.com
       ServerAdmin webmaster@your-domain.com
       DocumentRoot /path/to/cannaspot

       # Proxy Flask application
       ProxyPreserveHost On
       ProxyPass / http://127.0.0.1:8000/
       ProxyPassReverse / http://127.0.0.1:8000/

       # Static files handling
       Alias /static /path/to/cannaspot/static
       Alias /uploads /path/to/cannaspot/uploads

       <Directory /path/to/cannaspot/static>
           Require all granted
       </Directory>

       <Directory /path/to/cannaspot/uploads>
           Require all granted
           Options -Indexes
           # Limit file types that can be uploaded
           <FilesMatch "\.(jpg|jpeg|png|gif|mp4|webm)$">
               Require all granted
           </FilesMatch>
       </Directory>

       # Security headers
       Header set X-Content-Type-Options "nosniff"
       Header set X-Frame-Options "SAMEORIGIN"
       Header set X-XSS-Protection "1; mode=block"

       # Large file upload settings
       TimeOut 3600
       ProxyTimeout 3600
       LimitRequestBody 536870912  # 512MB

       ErrorLog ${APACHE_LOG_DIR}/cannaspot_error.log
       CustomLog ${APACHE_LOG_DIR}/cannaspot_access.log combined
   </VirtualHost>
   ```

4. Run application:
   ```bash
   gunicorn -w 4 -b 0.0.0.0:8000 app:app
   ```

### Production Configuration
- Set `debug=False` in `app.run()`
- Configure proper logging
- Set up SSL/TLS certificates
- Implement proper backup strategy for uploads directory
- Configure maximum upload size in web server config

### Prerequisites
- Python 3.6+
- MySQL (if not using SQLite)
- DirectAdmin control panel
- Apache with Python support
- WSGI server (e.g., Gunicorn)

### DirectAdmin Deployment Steps
1. Create a Python application in DirectAdmin:
   - Log into DirectAdmin
   - Go to "Custom Application Versions"
   - Add Python 3.6+ if not already available

2. Configure Python Environment:
   ```bash
   cd ~/domains/your-domain.com/public_html
   python3 -m venv venv
   source venv/bin/activate
   ```

3. Upload Application:
   - Use DirectAdmin File Manager or FTP to upload all files
   - Ensure proper permissions:
   ```bash
   chmod -R 755 ~/domains/your-domain.com/public_html
   chmod -R 777 uploads/
   ```

4. Create custom handlers in DirectAdmin:
   - Go to "Custom HTTPD Configurations"
   - Add the following to your domain's configuration:

   ```apache
   # Python WSGI Configuration
   WSGIDaemonProcess cannaspot python-path=/home/username/domains/your-domain.com/public_html/venv/lib/python3.x/site-packages
   WSGIProcessGroup cannaspot
   WSGIScriptAlias / /home/username/domains/your-domain.com/public_html/app.wsgi

   # Static and Upload Directories
   Alias /static /home/username/domains/your-domain.com/public_html/static
   Alias /uploads /home/username/domains/your-domain.com/public_html/uploads

   <Directory /home/username/domains/your-domain.com/public_html>
       Require all granted
       WSGIScriptReloading On
   </Directory>

   <Directory /home/username/domains/your-domain.com/public_html/uploads>
       Require all granted
       Options -Indexes
       <FilesMatch "\.(jpg|jpeg|png|gif|mp4|webm)$">
           Require all granted
       </FilesMatch>
   </Directory>
   ```

5. Create WSGI file:
   Create `app.wsgi` in your domain root:
   ```python
   import sys
   import os

   # Add application directory to path
   sys.path.insert(0, '/home/username/domains/your-domain.com/public_html')

   # Activate virtual environment
   activate_this = '/home/username/domains/your-domain.com/public_html/venv/bin/activate_this.py'
   with open(activate_this) as file_:
       exec(file_.read(), dict(__file__=activate_this))

   # Import and create application
   from app import app as application
   ```

### Regular Deployment Steps
1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   pip install gunicorn
   ```

2. Configure environment variables:
   - Create `.env` file in project root
   - Set `SECRET_KEY` for production
   - Set `DATABASE_URL` if using MySQL

3. Set up Apache web server:
   ```apache
   # Enable required modules
   LoadModule proxy_module modules/mod_proxy.so
   LoadModule proxy_http_module modules/mod_proxy_http.so
   LoadModule headers_module modules/mod_headers.so
   LoadModule rewrite_module modules/mod_rewrite.so

   <VirtualHost *:80>
       ServerName your-domain.com
       ServerAdmin webmaster@your-domain.com
       DocumentRoot /path/to/cannaspot

       # Proxy Flask application
       ProxyPreserveHost On
       ProxyPass / http://127.0.0.1:8000/
       ProxyPassReverse / http://127.0.0.1:8000/

       # Static files handling
       Alias /static /path/to/cannaspot/static
       Alias /uploads /path/to/cannaspot/uploads

       <Directory /path/to/cannaspot/static>
           Require all granted
       </Directory>

       <Directory /path/to/cannaspot/uploads>
           Require all granted
           Options -Indexes
           # Limit file types that can be uploaded
           <FilesMatch "\.(jpg|jpeg|png|gif|mp4|webm)$">
               Require all granted
           </FilesMatch>
       </Directory>

       # Security headers
       Header set X-Content-Type-Options "nosniff"
       Header set X-Frame-Options "SAMEORIGIN"
       Header set X-XSS-Protection "1; mode=block"

       # Large file upload settings
       TimeOut 3600
       ProxyTimeout 3600
       LimitRequestBody 536870912  # 512MB

       ErrorLog ${APACHE_LOG_DIR}/cannaspot_error.log
       CustomLog ${APACHE_LOG_DIR}/cannaspot_access.log combined
   </VirtualHost>
   ```

4. Run application:
   ```bash
   gunicorn -w 4 -b 0.0.0.0:8000 app:app
   ```

### Production Configuration
- Set `debug=False` in `app.run()`
- Configure proper logging
- Set up SSL/TLS certificates
- Implement proper backup strategy for uploads directory
- Configure maximum upload size in web server config

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davebowl)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/davebowl)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
