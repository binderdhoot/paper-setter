# Paper Setter

A single-page, no-build-step web app for assembling and printing school/college question papers, right in the browser.

Open [index.html](index.html) — there's nothing to install and no server required.

## What it does

- **Paper details** — set title, class, subject, max marks, duration, and date. A marks meter tracks your running total against the max marks as you add questions.
- **Five question types** — Multiple Choice (with 2–6 options and a marked correct answer), Fill in the Blank, Short Answer, Long Answer, and Passage-based questions with nested sub-questions (each sub-question can itself be MCQ, fill in the blank, short, or long).
- **Images** — attach an image (diagram, graph, photo) to any question, with an optional caption.
- **Question list** — reorder, edit, or delete questions; each entry shows its type, marks, and a text preview.
- **Live preview** — a formatted, printable paper sheet that updates as you build, with a toggle between the **question paper** view and the **answer key** view.
- **Export** — download a paginated **PDF** with consistent margins and "Page X of Y" numbering on every page, or a real, ready-to-open **.docx** file, both generated client-side with one click (no print dialog, no server round-trip).
- **Autosave** — everything is saved to `localStorage` as you type, so a refresh won't lose your work. Ships with sample content on first load, which you can dismiss or clear.
- **Responsive** — a two-pane build/preview layout on desktop, collapsing to a tabbed view on mobile.

## Tech

Plain HTML, CSS, and vanilla JavaScript in a single file — no framework, no build tooling. Loads a few things from a CDN:
- Google Fonts (Source Serif 4, IBM Plex Sans, IBM Plex Mono)
- [docx](https://docx.js.org) — generates genuine `.docx` files client-side (no server, no upload)
- [JSZip](https://stuk.github.io/jszip/) — re-shapes the generated `.docx`'s embedded-image XML to match what MS Word itself writes, since some real-world Word installs reject the perfectly schema-valid but non-standard markup `docx` produces by default
- [html2canvas](https://html2canvas.hertzen.com/) + [jsPDF](https://github.com/parallax/jsPDF) — render the paper to a paginated PDF directly, rather than through the browser's print dialog. Page breaks are computed from the actual layout (a question, option grid, or sub-question is never split across pages), and every page gets the same fixed margin and a "Page X of Y" footer — both of which turned out to be inconsistent across real browsers/OSes when left to `window.print()` and CSS `@page` rules alone.

Supports light and dark color schemes.

Favicon, app icon, and social-share image live in [assets/](assets/) (`favicon.png`, `app-icon.png`, `logo-mark.png`, `og-image.png`), derived from the two full-size reference sheets also in that folder.

## Usage

Just open `index.html` in a browser. Fill in the paper details, add questions from the "Add a question" panel, and use the preview pane on the right to check formatting and print.
