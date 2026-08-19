# KinoBot + TelePlay — FINAL IDEAL IMPLEMENTATION PROMPT v3

## 0. ROLE

You are the implementation engineer for this project.

Two existing projects are provided:

- `kinobot-project`
- `TelePlay-main`

Your task is to merge them into one production-ready Telegram KinoBot/WebApp.

IMPORTANT:
Before changing ANY code, inspect BOTH projects completely enough to understand their actual architecture, dependencies, API routes, database models, frontend screens, upload flow, authentication, Telegram integration, streaming implementation, and deployment configuration.

Do not assume that the descriptions in this prompt exactly match the source code. The source code is authoritative for implementation details. This prompt defines the TARGET architecture and behavior.

Do not delete or rewrite working functionality merely because another implementation would be cleaner. Reuse working code where it fits the target architecture, especially TelePlay's Telegram streaming implementation.

---

# 1. FINAL PRODUCT

Build one Telegram-based movie catalog service.

The final system must work as follows:

ADMIN
→ uploads a movie through Telegram bot or Admin WebApp
→ backend receives the video
→ video is stored in a private Telegram storage channel
→ database stores the Telegram storage message/file reference
→ admin enters movie metadata
→ movie becomes visible in the common catalog

USER
→ opens Telegram WebApp
→ authenticates automatically through Telegram WebApp initData
→ sees the SAME common movie catalog as every other user
→ opens a movie
→ backend verifies access rules
→ backend streams the movie directly from the private Telegram storage channel
→ browser `<video>` element plays the stream using HTTP Range requests
→ no movie file is stored on the application server
→ no Cloudflare R2 is used.

There is NO personal movie library.

All users see the administrator's common catalog.

---

# 2. SOURCE PROJECTS

## 2.1 TelePlay-main

Use TelePlay-main primarily as the backend/streaming foundation.

Inspect and reuse its actual working:

- FastAPI structure
- Telegram integration
- PyroTGFork/Telegram client logic
- Telegram media reading
- chunked streaming
- HTTP Range support
- `206 Partial Content`
- `Content-Range`
- `Accept-Ranges`
- database infrastructure where appropriate
- configuration/environment handling

The existing Telegram streaming implementation is considered a high-value component.

Do NOT replace it with an unrelated streaming architecture unless the existing implementation is demonstrably incompatible with the final requirements.

## 2.2 kinobot-project

Use kinobot-project primarily as the frontend/UI and business-function reference.

Preserve the existing frontend design and screens wherever possible.

Inspect and preserve:

- existing frontend screens
- movie catalog UI
- movie detail UI
- player UI
- favorites UI
- history/continue watching UI
- premium UI
- payment UI
- required subscription UI
- contact UI
- admin panel
- broadcast UI
- existing design system
- responsive behavior

The frontend visual design must NOT be unnecessarily redesigned.

The frontend must be adapted to the new FastAPI backend.

---

# 3. NON-NEGOTIABLE ARCHITECTURE

Final stack:

Backend:
- Python 3.11+
- FastAPI
- Uvicorn
- SQLAlchemy 2.x
- SQLite initially
- PyroTGFork/Telegram client from TelePlay where compatible

Frontend:
- existing kinobot-project frontend
- vanilla HTML/CSS/JS if that is what the source project uses
- preserve current UI/design

Telegram:
- Telegram Bot API for bot interaction where appropriate
- PyroTGFork/MTProto for private-channel media access and streaming where TelePlay already uses it

Deployment:
- Render.com

Storage:
- Movie binary files: PRIVATE TELEGRAM STORAGE CHANNEL
- Application server: NO movie-file storage
- Cloudflare R2: COMPLETELY REMOVED

---

# 4. FIRST PHASE — FULL CODE AUDIT

Before editing:

1. Inspect the entire directory structure of both projects.
2. Identify all backend entry points.
3. Identify all FastAPI routers.
4. Identify all Telegram handlers.
5. Identify all Telegram client/session logic.
6. Identify all streaming functions.
7. Identify all database models.
8. Identify all database initialization/migration logic.
9. Identify all frontend API calls.
10. Identify all player logic.
11. Identify all R2 code.
12. Identify all multi-quality video code.
13. Identify all authentication code.
14. Identify all admin authentication code.
15. Identify all premium/payment logic.
16. Identify required-subscription logic.
17. Identify broadcast logic.
18. Identify contact logic.
19. Identify audit logging.
20. Identify favorites/history.
21. Identify Render/deployment configuration.
22. Identify `.env` variables.
23. Identify existing `db.json` data and its exact structure.
24. Identify all dependencies.

Create an internal migration map before implementation.

Do not start blindly modifying files.

---

# 5. TARGET VIDEO STORAGE PIPELINE

The final pipeline MUST be:

Admin video
→ FastAPI/Telegram bot
→ private Telegram storage channel
→ Telegram message
→ database record
→ streaming endpoint
→ browser `<video>`

For every movie, store enough Telegram information to reliably locate the exact media.

At minimum the database should retain:

- storage channel ID
- storage message ID
- Telegram file ID if available
- duration if available
- file size if available
- MIME type if available

Do not store the movie binary in:

- local filesystem
- Render filesystem
- SQLite
- R2
- arbitrary object storage

---

# 6. ADMIN-ONLY VIDEO UPLOAD

This is mandatory.

Only the configured administrator may create/upload movies.

The backend MUST verify the Telegram user ID against `ADMIN_ID`.

A normal user sending a video/document/audio to the bot must NOT:

- create a movie
- upload media to the storage channel
- create a DB movie record
- gain admin functionality

Reject unauthorized media uploads safely.

Admin upload may happen through:

A. Telegram bot

or

B. Admin WebApp

Both must eventually use the same backend movie-creation pipeline.

Do not implement two unrelated storage mechanisms.

---

# 7. MOVIE CREATION TRANSACTION

Movie creation must be robust.

Recommended state flow:

`draft`
→ `uploading`
→ `stored`
→ `published`

If Telegram upload/storage fails:

- do not publish the movie
- do not leave an unusable movie record
- clean up partial state where possible
- report the failure to admin

Only a movie with a valid Telegram storage reference may become visible to users.

If metadata is entered before upload, keep the movie unpublished until storage succeeds.

---

# 8. DATABASE

Use SQLAlchemy 2.x.

Target conceptual schema:

## movies

- id
- title
- description
- year
- genre information
- poster_path
- banner_path
- storage_channel_id
- channel_message_id
- telegram_file_id
- mime_type
- file_size
- duration
- status
- is_premium
- created_at
- updated_at

## genres

- id
- name
- is_active

## users

- id
- telegram_id
- username
- first_name
- is_blocked
- created_at

## favorites

- user_id
- movie_id

Unique constraint:
`user_id + movie_id`

## watch_history

- user_id
- movie_id
- progress_seconds
- updated_at

Unique constraint:
`user_id + movie_id`

## premium_plans

- id
- name
- price
- duration_days
- is_active

## payments

- id
- user_id
- plan_id
- status
- proof_image
- created_at
- reviewed_by
- reviewed_at
- admin_note if required

## required_channels

- id
- channel_username
- channel_id
- is_active

## contact_messages

- id
- user_id
- text
- is_read
- created_at

## audit_log

- id
- admin_id
- action
- target
- created_at
- metadata if useful

## admin_settings

- key
- value

Add appropriate indexes and foreign keys.

Do not blindly copy this schema if the existing source structure provides a better equivalent. Preserve the target behavior and relationships.

---

# 9. DATABASE MIGRATION

Create a one-time migration utility:

`db.json`
→ SQLAlchemy SQLite database

Before writing the migration:

1. inspect the actual `db.json`;
2. inspect all actual JSON fields;
3. map each source field explicitly;
4. preserve existing users, movies, genres, favorites, history, plans and other meaningful data where applicable;
5. document fields that cannot be migrated.

The migration must be idempotent or safely repeatable.

Do not silently discard source data.

---

# 10. REMOVE CLOUDFLARE R2 COMPLETELY

Cloudflare R2 must disappear from the final project.

Remove:

- R2 SDK/dependencies
- R2 configuration
- R2 environment variables
- presigned URL generation
- R2 upload endpoints
- R2 delete endpoints
- R2 video URL logic
- R2 multi-quality storage
- frontend R2 upload logic
- frontend R2 player logic
- dead R2 helper modules

Search the ENTIRE repository for:

- R2
- Cloudflare
- S3
- presigned
- signed URL
- bucket
- endpoint references related to R2

Do not remove unrelated S3-compatible code unless it is specifically part of the old R2 movie-storage implementation.

---

# 11. REMOVE MULTI-QUALITY VIDEO SYSTEM

The final product uses exactly ONE video file per movie.

No:

- 360p
- 480p
- 720p
- 1080p source selection
- quality dropdown
- quality variants
- multiple Telegram messages for different qualities

The player should request one stream:

`/api/movies/{movie_id}/stream`

The backend determines the actual Telegram source.

---

# 12. STREAMING ENDPOINT

Implement/retain a production-quality endpoint similar to:

`GET /api/movies/{movie_id}/stream`

The endpoint must:

1. authenticate the Telegram WebApp user;
2. verify the movie exists;
3. verify the movie is published;
4. verify required-channel subscription if enabled;
5. verify premium access if the movie is premium;
6. locate the Telegram storage message;
7. support HTTP Range requests;
8. return correct partial-content responses;
9. stream incrementally from Telegram;
10. never download the entire movie into RAM;
11. never save the movie to disk;
12. support browser seeking;
13. support pause/resume;
14. support progressive playback.

Required HTTP behavior should include appropriate headers such as:

- `Accept-Ranges: bytes`
- `Content-Range`
- `Content-Length`
- correct `Content-Type`
- `206 Partial Content` for valid range requests

Handle:

- no Range header
- valid Range
- invalid Range
- multiple sequential Range requests
- client disconnect
- Telegram errors
- missing/deleted storage message

Do not buffer the entire movie before responding.

---

# 13. STREAM SECURITY

The stream endpoint MUST NOT be publicly usable without access control.

Do not expose the Telegram storage channel publicly.

Do not return the Telegram storage channel's public URL to users.

The browser should receive the application backend stream endpoint.

Access must be checked server-side.

Do not trust frontend JavaScript for:

- premium access
- subscription status
- admin status

---

# 14. TELEGRAM WEBAPP AUTHENTICATION

Normal users:

Use Telegram WebApp `initData`.

Validate it server-side according to Telegram's WebApp authentication requirements.

Use HMAC-SHA256 validation.

Do NOT use:

- JWT
- login codes
- remote authentication
- unnecessary password login

unless some completely separate admin-only mechanism requires it.

The authenticated Telegram user should be mapped to the `users` table.

Do not trust user ID values sent only in ordinary JSON payloads.

---

# 15. ADMIN AUTHENTICATION

Admin access requires BOTH:

1. valid Telegram WebApp authentication;
2. configured admin credential/key.

The admin credential must never be exposed to normal users.

Admin endpoints must independently verify admin authorization.

Do not rely on hiding frontend buttons.

Provide an admin credential change mechanism if the existing product already supports it.

Store secrets securely.

Never hard-code:

- bot token
- API ID
- API hash
- admin key
- channel ID

---

# 16. FRONTEND

Preserve the existing kinobot-project frontend design.

Do not redesign the 12 existing screens without a concrete requirement.

Update only what is necessary to connect the frontend to FastAPI.

At minimum inspect/update:

- `frontend/js/api.js`
- `frontend/js/player.js`
- movie catalog requests
- movie detail requests
- favorites
- history
- premium
- payments
- required subscription
- contact
- admin upload
- admin movie management
- broadcast
- authentication

Replace old Node/R2 API calls with FastAPI endpoints.

---

# 17. PLAYER

The final player must use ONE source.

Conceptually:

`<video src="/api/movies/{id}/stream">`

or an equivalent authenticated request mechanism compatible with Telegram WebApp authentication.

The player must support:

- play
- pause
- seek
- browser buffering
- resume
- progress tracking
- mobile browser playback

Do not implement client-side Telegram media downloading.

Do not expose Telegram credentials.

---

# 18. POSTER/BANNER STORAGE

Because R2 is removed, inspect the existing frontend/backend implementation and decide on a practical final storage mechanism for posters/banners.

Do NOT silently leave poster/banners pointing to removed R2 URLs.

Possible implementation may use:

- Telegram storage
- application-managed small image files if appropriate
- another explicitly configured storage system

The chosen mechanism must be documented and implemented consistently.

Movie video storage and poster/banner storage do not have to be identical, but movie binaries MUST remain in Telegram.

---

# 19. PREMIUM

Preserve the existing premium business logic from kinobot-project.

Implement it in FastAPI/SQLAlchemy.

Required behavior:

- admin creates/manages plans;
- user selects a plan;
- user can submit payment proof;
- admin can approve/reject;
- approved premium status grants access;
- expired premium status does not grant access;
- premium movie access is checked server-side.

A premium movie must not be accessible merely because its frontend card is visible.

---

# 20. REQUIRED SUBSCRIPTION

Preserve the existing mandatory-subscription functionality.

Before playback of a restricted movie:

- backend verifies subscription;
- if user is not subscribed, playback is denied;
- frontend displays the subscription requirement.

Never rely only on frontend checks.

Support multiple required channels if the existing product supports them.

---

# 21. FAVORITES

Preserve favorites.

User can:

- add movie to favorites;
- remove movie;
- list favorites.

Use DB constraints to prevent duplicate favorites.

---

# 22. WATCH HISTORY / CONTINUE WATCHING

Preserve:

- watch history
- progress
- continue watching

Store progress in seconds.

Update progress periodically and/or on relevant player events.

Do not send a request for every video frame.

Use sensible throttling/debouncing.

---

# 23. CONTACT

Preserve user → admin contact functionality.

Admin must be able to see/read contact messages.

Record messages in DB.

---

# 24. BROADCAST

Preserve admin broadcast.

Only admin may initiate broadcasts.

Handle Telegram failures gracefully.

Do not let one blocked/deleted Telegram account terminate the entire broadcast.

Record useful audit information.

---

# 25. AUDIT LOG

Log important admin actions, including at minimum:

- admin login/access
- movie creation
- movie publication
- movie deletion
- movie update
- premium approval
- premium rejection
- plan changes
- required-channel changes
- broadcast
- settings changes

Do not log secrets.

---

# 26. MOVIE DELETE

When an admin deletes a movie:

1. remove/hide the DB movie;
2. remove its related favorites/history as appropriate;
3. delete the corresponding Telegram storage message if safely possible;
4. record the action in audit log.

If Telegram deletion fails, do not pretend it succeeded.

Handle orphaned Telegram messages safely.

---

# 27. API DESIGN

Create a coherent FastAPI API.

Examples:

Authentication:
- `GET/POST /api/auth/...`

Movies:
- `GET /api/movies`
- `GET /api/movies/{id}`
- `POST /api/movies`
- `PATCH /api/movies/{id}`
- `DELETE /api/movies/{id}`
- `GET /api/movies/{id}/stream`

Favorites:
- `GET /api/favorites`
- `POST /api/favorites/{movie_id}`
- `DELETE /api/favorites/{movie_id}`

History:
- `GET /api/history`
- `POST /api/history/{movie_id}`

Premium:
- plans
- payment submission
- admin review

Required subscription:
- status/check endpoint

Contact:
- user contact
- admin management

Admin:
- dashboard
- movie management
- users
- payments
- broadcast
- settings
- audit logs

These are examples, not an excuse to create duplicate or unnecessary routes.

First inspect existing frontend API calls and then design the final routes so the frontend can be migrated cleanly.

---

# 28. ERROR HANDLING

Never expose:

- Telegram API credentials
- stack traces
- internal filesystem paths
- database details
- private channel information

Use appropriate HTTP status codes.

Return consistent JSON error structures.

Log detailed technical errors server-side.

Return user-friendly messages to the frontend.

---

# 29. CONCURRENCY / TELEGRAM STREAMING

The system must support multiple users watching movies simultaneously.

Inspect TelePlay's current Telegram client/session implementation.

Do not create a new Telegram client/session for every HTTP request unless the existing architecture explicitly requires it.

Reuse connections safely.

Avoid blocking the FastAPI event loop.

Ensure Telegram streaming does not cause the whole web service to freeze.

If the existing TelePlay implementation uses synchronous operations, adapt them safely to the FastAPI architecture.

---

# 30. RENDER DEPLOYMENT

Final deployment target:

Render.com Python Web Service.

The service must start correctly from a clean environment.

Environment variables should include only what is actually needed, for example:

- `TELEGRAM_API_ID`
- `TELEGRAM_API_HASH`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_STORAGE_CHANNEL_ID`
- `ADMIN_ID`
- `ADMIN_KEY`
- `DATABASE_URL`
- `WEBAPP_URL`
- other required Telegram/session configuration

Do NOT keep R2 variables.

IMPORTANT:

SQLite on Render requires persistent storage if database persistence across redeploy/restart is expected.

Do not falsely claim SQLite is persistent on Render without configuring an appropriate persistent disk.

If the existing deployment architecture requires a persistent Render disk, document the exact mount path.

The application must be deployable from a clean checkout.

---

# 31. DEPENDENCY CLEANUP

After implementation:

- remove unused Node backend dependencies if Node is no longer required;
- remove GramJS;
- remove R2 dependencies;
- remove unused React/TypeScript/Android/TelePlay frontend components if they are not part of the final product;
- remove obsolete authentication packages;
- remove dead code.

Do not delete frontend dependencies still required by the existing kinobot frontend.

Run dependency/import checks after cleanup.

---

# 32. TESTING

Do not stop after writing code.

Run real tests.

At minimum verify:

## Authentication
- valid Telegram WebApp user
- invalid initData
- expired/invalid auth data
- normal user
- admin user
- wrong admin key

## Movie management
- admin creates movie
- unauthorized user cannot create movie
- video reaches storage channel
- DB contains correct Telegram reference
- movie publishes only after storage succeeds
- admin edits movie
- admin deletes movie

## Streaming
- movie plays
- Range request works
- `206` works
- `Content-Range` works
- seeking works
- pause/resume works
- multiple users can stream
- unauthorized user cannot stream
- premium restriction works
- required-subscription restriction works
- missing Telegram message fails gracefully

## User features
- favorites
- history
- continue watching
- premium
- payment proof
- contact
- required subscription

## Admin
- broadcast
- audit log
- settings

## Migration
- `db.json` migration works
- existing data remains valid

---

# 33. FRONTEND REGRESSION TEST

Open/use the final WebApp and verify every existing screen.

Do not consider the project complete merely because the backend starts.

Verify:

- no broken API calls
- no console errors caused by migration
- movie posters load
- movie lists load
- detail page works
- player works
- favorites work
- history works
- premium works
- payment works
- subscription requirement works
- contact works
- admin panel works
- upload works
- broadcast works

---

# 34. SECURITY REVIEW

Before finalizing, search the whole project for:

- hard-coded secrets
- Telegram tokens
- API hashes
- admin passwords
- exposed storage channel identifiers where inappropriate
- debug endpoints
- unauthenticated admin endpoints
- unauthenticated stream endpoints
- direct Telegram media URLs
- R2 URLs
- old login/JWT systems
- development-only bypasses

Remove all production security bypasses.

---

# 35. IMPORTANT — DO NOT MAKE THESE MISTAKES

DO NOT:

- keep R2 “just in case”;
- keep multi-quality videos;
- allow every Telegram user to upload movies;
- create a separate personal library for every user;
- expose Telegram storage channel URLs;
- download full movies to disk before streaming;
- buffer entire movies in RAM;
- trust frontend premium checks;
- trust frontend admin checks;
- keep old JWT/login-code authentication;
- leave dead R2 API routes;
- leave frontend calls pointing to old Node/R2 APIs;
- rewrite TelePlay's working streaming system without reason;
- redesign the existing kinobot frontend unnecessarily;
- silently discard `db.json` data;
- claim deployment is persistent if SQLite is on ephemeral storage.

---

# 36. IMPLEMENTATION ORDER

Follow this order:

### Phase 1
Full source audit.

### Phase 2
Create final project architecture.

### Phase 3
Port/adapt TelePlay backend and Telegram streaming.

### Phase 4
Create SQLAlchemy models.

### Phase 5
Create and test `db.json` migration.

### Phase 6
Implement admin-only Telegram movie upload/storage pipeline.

### Phase 7
Implement movie CRUD.

### Phase 8
Implement secure Range streaming.

### Phase 9
Port Telegram WebApp authentication.

### Phase 10
Connect existing kinobot frontend.

### Phase 11
Port favorites/history.

### Phase 12
Port premium/payment.

### Phase 13
Port required subscription.

### Phase 14
Port contact/broadcast/audit.

### Phase 15
Remove R2/multi-quality/obsolete systems.

### Phase 16
Run complete tests.

### Phase 17
Fix all errors.

### Phase 18
Prepare Render deployment.

### Phase 19
Perform production-style smoke test.

---

# 37. DEFINITION OF DONE

The project is COMPLETE only when all of these are true:

- Backend starts successfully.
- Bot starts successfully.
- Admin can upload a movie.
- Only admin can upload movies.
- Movie is stored in the private Telegram channel.
- DB stores the correct Telegram storage reference.
- Movie appears in the common catalog.
- Normal user can authenticate through Telegram WebApp.
- Normal user can watch an allowed movie.
- Video streams without downloading the full movie to the server.
- HTTP Range works.
- Seeking works.
- Multiple users can stream.
- Premium access works.
- Required subscription works.
- Favorites work.
- History works.
- Continue watching works.
- Payment proof works.
- Admin approval/rejection works.
- Contact works.
- Broadcast works.
- Audit log works.
- Admin panel works.
- R2 is completely gone.
- Multi-quality is completely gone.
- Personal library is completely gone.
- JWT/login-code/remote-auth legacy system is gone.
- Old Node/GramJS backend is not required.
- Render deployment configuration works.
- No known critical errors remain.

---

# 38. FINAL RESPONSE AFTER IMPLEMENTATION

After completing the implementation, report:

1. What was changed.
2. Which source components were reused.
3. Which obsolete components were removed.
4. Final database schema.
5. Final API structure.
6. Telegram storage/streaming architecture.
7. Authentication architecture.
8. Tests performed and their results.
9. Any remaining limitations.
10. Exact Render environment variables required.
11. Exact deployment/start command.
12. Any manual steps the administrator must perform in Telegram.

Do not claim a feature is working unless it was actually tested.

If something cannot be tested in the current environment, explicitly state that it was not tested.

---

# FINAL PRIORITY

When requirements conflict, use this priority:

1. Security
2. Correct Telegram streaming
3. Correct admin-only movie storage
4. Data integrity
5. Existing kinobot UI/functionality
6. Performance
7. Code cleanliness

Do not optimize for speed of implementation at the cost of correctness.

The goal is NOT merely to merge two repositories.

The goal is to produce one coherent, secure, testable, production-ready KinoBot whose movie binaries live in a private Telegram channel and whose users watch them through the existing WebApp UI via backend Range streaming.
