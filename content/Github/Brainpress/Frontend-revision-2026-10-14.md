# Frontend revision — 2026-10-14

## Confirmed direction

- Brainpress remains independently developed with a single post model and flat categories.
- Reference WordPress content-management conventions and the actual Zibll forum frontend rather than inventing a new visual direction.
- Use English for project UI, documentation, and new implementation text. Existing authored content is not automatically translated.
- Add five mobile bottom navigation entries: Home, AI chat, Publish, Messages, Profile. AI chat, Messages, and Profile remain placeholders.
- Add Mermaid, LaTeX mathematics, common Markdown extensions, and light/dark modes.

## Reference material inspected

- User screenshot note: `zibll论坛截图.md`.
- Five linked screenshots from the wiki Attachments directory.
- Zibll `inc/functions/bbs/assets/css/main.css`, especially forum card padding, title weight, line height, author and footer hierarchy.
- Zibll `js/main.js` theme toggle: View Transition API, circular reveal from the triggering button, and a direct-toggle fallback.

## Implemented layout

- Desktop: narrow left navigation, center notice/quick publishing/feed cards, right profile/category cards.
- Mobile: centered logo, menu/search controls, horizontal feed/category navigation, and five fixed bottom entries.
- Dedicated publishing: left Markdown editor and image insertion area, right category/publish controls.
- Forum cards: author header, timestamp, title, excerpt, optional image, category badge, and share action.
- Light and dark color tokens cover navigation, editor, cards, tables, dialogs, and placeholders. Theme defaults to the system preference until explicitly selected, then persists locally.
- Circular 500ms theme transition when supported; reduced-motion users get a direct change. Small hover, dialog, and card transitions follow the reference interaction style.

## Markdown support

CommonMark plus GFM tables, task lists, strikethrough, automatic links, and footnotes. Mermaid fenced code becomes a diagram. KaTeX renders `$...$`, `$$...$$`, `\(...\)`, and `\[...\]` formulas. Math is parsed separately so underscores and backslashes remain intact. Code spans and fences stay literal. Raw Markdown remains the database source.

Libraries are bundled locally; rendering does not depend on a CDN. Original HTML remains escaped, links are filtered, Mermaid uses strict mode, and KaTeX trust is disabled.

## Validation

- PHP and JavaScript syntax checks.
- Database source preservation, category integrity, safe rendering, GFM, footnotes, math delimiters, Unicode context, ordinary prices, and literal code tests.
- Browser confirmed two Mermaid diagrams and five formulas in both the reading page and repeated editor previews.
- Saved an edit while in preview mode successfully.
- Theme choice survived a page reload.
- Responsive checks at desktop and 402px mobile widths; mobile tabs visible and no horizontal page overflow.
- Homepage, dedicated editor, categories, search, and all placeholder routes returned successful responses.

English demo: http://127.0.0.1:8081/?post=3
Local app: http://127.0.0.1:8081/

MySQL remains unverified on a real server. User accounts and actual AI/messaging/profile functions are outside this revision.
