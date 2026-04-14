Listings Marketplace – GitHub Copilot Instructions
Project Overview
Project Name: Listings Marketplace (Обяви с картинки)
Goal: A multi-page, full-stack web application where users can post, browse, and manage listings with image uploads. Admins can moderate content. Built as a Soft-Tech-with-AI Capstone Project.

Tech Stack
LayerTechnologyFrontendVanilla JavaScript (ES Modules), HTML5, CSS3UI FrameworkBootstrap 5 (CDN or npm)Build ToolVite (multi-page mode)Backend / BaaSSupabaseDatabasePostgreSQL (via Supabase)AuthenticationSupabase Auth (email/password)File StorageSupabase StorageAPISupabase REST API + JS SDK (@supabase/supabase-js)HostingNetlifySource ControlGitHubAI AssistantGitHub Copilot inside VS Code

Strict Rules – Follow at All Times

No frameworks – Do NOT use React, Vue, Angular, Svelte, or TypeScript. Vanilla JS only.
Multi-page architecture – Every page is a separate .html file. Do NOT build a SPA with client-side routing.
Modular JS – Each page loads its own src/pages/*.js module. Shared logic lives in src/services/, src/components/, or src/utils/.
Security first – ALL data access rules are enforced via Supabase Row-Level Security (RLS) policies in the database. Never rely on UI-only checks for security.
Admin logic belongs in RLS + DB – The is_admin() SQL function (security definer) must be the single source of truth for admin checks.
All DB calls go through src/services/ – Pages never import supabaseClient directly for queries; they call service functions.
Environment variables – Supabase URL and anon key are stored in .env (Vite prefix VITE_). Never hardcode credentials.
Mobile-first – Use Bootstrap grid and utility classes. Every page must be usable on a 375px screen.
Consistent error handling – All async service calls are wrapped in try/catch. Errors surface via src/components/toast.js.


Folder Structure
listings-marketplace/
├── .github/
│   └── copilot-instructions.md     ← this file
├── .env                             ← VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY
├── .env.example
├── .gitignore
├── vite.config.js                   ← multi-page input config
├── package.json
├── README.md
│
├── index.html                       ← Public listing feed + search/filter
├── register.html                    ← User registration
├── login.html                       ← User login
├── listing-create.html              ← Create new listing + image upload
├── listing-edit.html                ← Edit listing + manage images
├── listing-details.html             ← Listing detail + comments + image download
├── profile.html                     ← User profile + avatar upload
├── admin.html                       ← Admin panel (users + listings moderation)
│
└── src/
    ├── pages/
    │   ├── index.js
    │   ├── register.js
    │   ├── login.js
    │   ├── listing-create.js
    │   ├── listing-edit.js
    │   ├── listing-details.js
    │   ├── profile.js
    │   └── admin.js
    │
    ├── services/
    │   ├── supabaseClient.js        ← createClient() singleton
    │   ├── authService.js           ← register, login, logout, getSession
    │   ├── listingsService.js       ← CRUD for listings
    │   ├── commentsService.js       ← CRUD for comments
    │   ├── storageService.js        ← upload/download/delete images & avatars
    │   ├── profileService.js        ← read/update profile, avatar
    │   └── adminService.js          ← admin: user list, change roles, moderate
    │
    ├── components/
    │   ├── navbar.js                ← renders nav, highlights active page, auth state
    │   ├── toast.js                 ← showToast(message, type)
    │   └── loader.js                ← showLoader() / hideLoader()
    │
    ├── utils/
    │   ├── guard.js                 ← requireAuth(), requireAdmin(), redirectIfLoggedIn()
    │   └── format.js                ← formatDate(), formatPrice(), truncate()
    │
    └── styles/
        └── app.css                  ← global overrides on top of Bootstrap

Database Schema
Table: profiles
sqlid          uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE
display_name text
avatar_url  text
created_at  timestamptz DEFAULT now()
Table: user_roles
sqluser_id  uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE
role     text NOT NULL CHECK (role IN ('user', 'admin')) DEFAULT 'user'
Table: listings
sqlid          uuid PRIMARY KEY DEFAULT gen_random_uuid()
owner_id    uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE
title       text NOT NULL
description text
price       numeric(10, 2)
status      text NOT NULL CHECK (status IN ('draft', 'published', 'archived')) DEFAULT 'draft'
created_at  timestamptz DEFAULT now()
Table: listing_images
sqlid          uuid PRIMARY KEY DEFAULT gen_random_uuid()
listing_id  uuid NOT NULL REFERENCES listings(id) ON DELETE CASCADE
owner_id    uuid NOT NULL REFERENCES auth.users(id)
file_path   text NOT NULL
public_url  text NOT NULL
created_at  timestamptz DEFAULT now()
Table: comments
sqlid          uuid PRIMARY KEY DEFAULT gen_random_uuid()
listing_id  uuid NOT NULL REFERENCES listings(id) ON DELETE CASCADE
author_id   uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE
content     text NOT NULL
created_at  timestamptz DEFAULT now()
ER Diagram (Mermaid)
mermaiderDiagram
    profiles {
        uuid id PK
        text display_name
        text avatar_url
        timestamptz created_at
    }
    user_roles {
        uuid user_id PK
        text role
    }
    listings {
        uuid id PK
        uuid owner_id FK
        text title
        text description
        numeric price
        text status
        timestamptz created_at
    }
    listing_images {
        uuid id PK
        uuid listing_id FK
        uuid owner_id FK
        text file_path
        text public_url
        timestamptz created_at
    }
    comments {
        uuid id PK
        uuid listing_id FK
        uuid author_id FK
        text content
        timestamptz created_at
    }

    profiles ||--o{ listings : "owns"
    profiles ||--o{ comments : "writes"
    listings ||--o{ listing_images : "has"
    listings ||--o{ comments : "receives"
    profiles ||--o| user_roles : "has role"

Storage Buckets
BucketPath patternAccessavatars{userId}/avatar.pngAuthenticated read; owner writelisting-images{userId}/{listingId}/{uuid}.jpgPublic read (published); owner/admin write

RLS Policies
Helper Function (run once in Supabase SQL editor)
sqlCREATE OR REPLACE FUNCTION is_admin()
RETURNS boolean
LANGUAGE sql
SECURITY DEFINER
AS $$
  SELECT EXISTS (
    SELECT 1 FROM user_roles
    WHERE user_id = auth.uid() AND role = 'admin'
  );
$$;
profiles
sql-- SELECT: everyone
CREATE POLICY "profiles_select_all" ON profiles FOR SELECT USING (true);
-- INSERT: only own row
CREATE POLICY "profiles_insert_own" ON profiles FOR INSERT WITH CHECK (auth.uid() = id);
-- UPDATE: only own row
CREATE POLICY "profiles_update_own" ON profiles FOR UPDATE USING (auth.uid() = id);
user_roles
sql-- SELECT: own row only
CREATE POLICY "roles_select_own" ON user_roles FOR SELECT USING (auth.uid() = user_id);
-- ALL: admin only
CREATE POLICY "roles_admin_all" ON user_roles FOR ALL USING (is_admin());
listings
sql-- SELECT: published OR own OR admin
CREATE POLICY "listings_select" ON listings FOR SELECT USING (
  status = 'published' OR owner_id = auth.uid() OR is_admin()
);
-- INSERT: authenticated, own
CREATE POLICY "listings_insert" ON listings FOR INSERT WITH CHECK (auth.uid() = owner_id);
-- UPDATE: own OR admin
CREATE POLICY "listings_update" ON listings FOR UPDATE USING (
  owner_id = auth.uid() OR is_admin()
);
-- DELETE: own OR admin
CREATE POLICY "listings_delete" ON listings FOR DELETE USING (
  owner_id = auth.uid() OR is_admin()
);
listing_images
sql-- SELECT: follows listing visibility
CREATE POLICY "images_select" ON listing_images FOR SELECT USING (
  EXISTS (
    SELECT 1 FROM listings l
    WHERE l.id = listing_id
      AND (l.status = 'published' OR l.owner_id = auth.uid() OR is_admin())
  )
);
-- INSERT/DELETE: owner or admin
CREATE POLICY "images_write" ON listing_images FOR INSERT WITH CHECK (
  auth.uid() = owner_id OR is_admin()
);
CREATE POLICY "images_delete" ON listing_images FOR DELETE USING (
  auth.uid() = owner_id OR is_admin()
);
comments
sql-- SELECT: all authenticated
CREATE POLICY "comments_select" ON comments FOR SELECT USING (auth.role() = 'authenticated');
-- INSERT: authenticated
CREATE POLICY "comments_insert" ON comments FOR INSERT WITH CHECK (auth.uid() = author_id);
-- DELETE: own or admin
CREATE POLICY "comments_delete" ON comments FOR DELETE USING (
  auth.uid() = author_id OR is_admin()
);

Pages & Responsibilities
index.html + src/pages/index.js

Fetch and display all published listings (public, no auth required)
Search by title (client-side filter or Supabase ilike)
Filter by price range
Each card links to listing-details.html?id=...
Navbar shows Login/Register if not logged in; shows profile + logout if logged in

register.html + src/pages/register.js

Form: display_name, email, password, confirm password
Call authService.register() → also insert into profiles and user_roles
Redirect to index.html on success
guard.redirectIfLoggedIn() at page load

login.html + src/pages/login.js

Form: email, password
Call authService.login()
Redirect to index.html on success
guard.redirectIfLoggedIn() at page load

listing-create.html + src/pages/listing-create.js

guard.requireAuth() at page load
Form: title, description, price, status (draft/published), image file input (multiple)
listingsService.create() → returns new listing id
storageService.uploadListingImages(listingId, files) → insert into listing_images
Redirect to listing-details.html?id=...

listing-edit.html + src/pages/listing-edit.js

guard.requireAuth() at page load
Load listing by ?id= query param; verify owner or admin
Edit fields + change status
Show existing images with delete button
Upload new images
listingsService.update() + storageService.deleteImage() / storageService.uploadListingImages()

listing-details.html + src/pages/listing-details.js

Public page (no auth required to view published listings)
Load listing + images + comments by ?id=
Image gallery with download button (generates signed URL via storageService.getDownloadUrl())
Comments section: show all, add comment (requires auth), delete own comment
Owner/admin sees Edit and Delete buttons

profile.html + src/pages/profile.js

guard.requireAuth() at page load
Show and update display_name
Upload/replace avatar → storageService.uploadAvatar() → update profiles.avatar_url
List own listings with links

admin.html + src/pages/admin.js

guard.requireAdmin() at page load
Tab 1 – Users: list all profiles + roles, promote/demote to admin
Tab 2 – Listings: list all listings regardless of status, change status, delete


Service API Contracts
authService.js
jsregister(email, password, displayName) → { user, error }
login(email, password)                  → { session, error }
logout()                                → void
getSession()                            → session | null
getCurrentUser()                        → user | null
listingsService.js
jsgetPublished(search, minPrice, maxPrice) → listings[]
getById(id)                              → listing
getByOwner(userId)                       → listings[]
getAll()                                 → listings[]  // admin only
create(data)                             → listing
update(id, data)                         → listing
remove(id)                               → void
storageService.js
jsuploadAvatar(userId, file)                          → publicUrl
uploadListingImages(userId, listingId, files)       → { file_path, public_url }[]
deleteImage(filePath, bucket)                        → void
getDownloadUrl(filePath, bucket)                     → signedUrl
profileService.js
jsgetProfile(userId)           → profile
updateProfile(userId, data)  → profile
commentsService.js
jsgetByListing(listingId)            → comments[]
add(listingId, authorId, content)  → comment
remove(id)                         → void
adminService.js
jsgetAllUsers()                          → { profile, role }[]
setRole(userId, role)                  → void
getAllListings()                        → listings[]
setListingStatus(listingId, status)    → void
deleteListing(listingId)               → void

vite.config.js Template
jsimport { defineConfig } from 'vite'
import { resolve } from 'path'

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main:            resolve(__dirname, 'index.html'),
        login:           resolve(__dirname, 'login.html'),
        register:        resolve(__dirname, 'register.html'),
        listingCreate:   resolve(__dirname, 'listing-create.html'),
        listingEdit:     resolve(__dirname, 'listing-edit.html'),
        listingDetails:  resolve(__dirname, 'listing-details.html'),
        profile:         resolve(__dirname, 'profile.html'),
        admin:           resolve(__dirname, 'admin.html'),
      }
    }
  }
})

Development Workflow & Commit Plan
Day 1 – Foundation
#Commit message1init: Vite + Bootstrap multi-page setup, all HTML shells2feat: supabase client + env config3feat: register & login pages with authService4feat: auth guards (requireAuth, requireAdmin, redirectIfLoggedIn)5feat: DB schema v1 – tables + RLS policies + is_admin()6feat: profile page – read/update display_name
Day 2 – Core Features
#Commit message7feat: listings index – public feed with search & filter8feat: listing-create – form + image upload to storage9feat: listing-details – gallery, download, comments10feat: listing-edit – update fields, add/remove images11feat: avatar upload in profile page12fix: responsive polish, toasts, loaders
Day 3 – Admin, Security & Deploy
#Commit message13feat: admin panel – users list + role management14feat: admin panel – listing moderation (status + delete)15chore: Netlify deploy + env vars + smoke test16docs: README – setup, architecture, DB diagram, demo creds

README Checklist (minimum content)

 What the app does + roles (user / admin)
 Tech stack table
 Screens list (8 pages)
 DB schema + Mermaid ER diagram
 Local setup: npm install → copy .env.example → npm run dev
 Live URL (Netlify)
 Demo credentials: user@demo.com / demo123 and admin@demo.com / admin123


Key Conventions for Copilot

Always await Supabase calls and destructure { data, error }
Check if (error) throw error immediately after every Supabase call
Use supabase.auth.getSession() to protect pages; redirect to login.html if no session
Use URLSearchParams to read query params (?id=...) in detail/edit pages
Bootstrap classes only for layout/components; put custom CSS in src/styles/app.css
Toast notifications for all success/error feedback (never alert())
All date formatting via formatDate() in src/utils/format.js
Images: accept image/jpeg,image/png,image/webp; max 5 MB per file
Storage paths must include userId as first segment (required by RLS on buckets)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DMY76657) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
