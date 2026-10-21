# Publishing and reading update — 2026-10-14

User decisions: defer export/backup until last; no general database migration framework now; replace browser draft recovery with persisted Draft/Published states; use YAML Frontmatter metadata; put contents in the article sidebar following Zibll; copy buttons for code; prepare a separate clean deployable build; soften focus styles and unify selects.

Implemented: draft list, draft filtering and publish transitions; typed YAML parsing, author/date/title fallback, draft synchronization and unknown-key preservation; desktop sidebar contents/mobile collapsible contents; code copy; subtle form controls. Editor title/status override corresponding metadata when supplied. Existing draft property is synchronized; changing it may reformat YAML comments. No file import/export yet. No full Obsidian compatibility claim.

Production package contains PHP dependencies, browser assets and Nginx/Apache examples, excludes personal DB/uploads/config and development runtimes. Website root must be public/. No login: protect personal cloud tests with HTTP Basic Auth or IP restriction. MySQL live testing remains deferred.

Also completed lightweight Manage: post trash/restore preserving prior status, category rename, deletion with reassignment including drafts and trash. Integration tests and clean package startup passed.
