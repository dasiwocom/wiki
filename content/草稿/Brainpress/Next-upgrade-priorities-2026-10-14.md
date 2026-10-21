# Next upgrade priorities — 2026-10-14

Status: assistant recommendations after reviewing the current implementation; not yet approved as implementation scope.

## 1. Complete content management

Posts can currently be created and edited, but there is no trash/restore flow. Categories can only be created. Add a small Manage page for posts and categories, post trash/restore, category renaming, and explicit reassignment when removing a category that contains posts. Keep the existing single post model and flat categories.

## 2. Protect writing from accidental loss

The editor warns before leaving but has no saved draft recovery. Start with browser-local draft autosaving and restoration for both quick publishing and dedicated editing. Handle successful publish cleanup and distinguish drafts for different posts.

## 3. Prepare cloud testing

Verify on the target PHP/MySQL environment; provide deployment configuration, a minimal environment check, explicit installation/schema upgrade steps, and an upload package. The application currently runs CREATE TABLE IF NOT EXISTS on each new database connection; this does not implement future schema upgrades. SQLite-to-MySQL migration is needed only if preserving the existing local data while switching databases.

If cloud access is public, add a minimal owner-only protection for write/upload operations or restrict the test site externally. This is a deployment-stage decision, not a proposal to add a multi-user system now.

## 4. Make content portable

Add Markdown export first, then an archive containing Markdown, metadata, and referenced uploaded images. Provide a database-and-images backup workflow. Image paths and category/title/date metadata must survive restoration.

## 5. Refine reading and uploads

Add heading anchors and an article table of contents, code-copy controls, image enlargement, and better loading/error feedback. Support image drag/drop and paste with insertion at the cursor. Compress oversized images and produce thumbnails before expanding into a full media library.

## Later work

Avoid prioritizing AI, messages, profiles, complex taxonomies, or plugin infrastructure before the personal publishing workflow is complete. Rendering caches, stored excerpts, indexes, and search upgrades should follow measurements or content growth. The current homepage reparses article bodies for excerpts, and category/post counts are calculated on page requests.

Recommended next development slice: a minimal Manage page plus browser-local draft recovery. This improves daily usability without expanding into a full administration or account system.
