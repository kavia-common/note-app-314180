# Notes App 314180 – Overview

## Overview

Notes App 314180 is a small, static, single-page notes application that runs entirely in the browser. It lets users create, edit, and delete notes written in Markdown. Notes are rendered to HTML using Marked.js and are persisted in the browser’s `localStorage`, so they are restored automatically the next time the page is loaded in the same browser.

## Features

### Core Features

1. Create new notes via an Add Note button.
2. Edit notes using a textarea with live Markdown preview (powered by Marked.js).
3. Delete notes individually from the page and from persisted storage.
4. Automatic persistence of all notes to browser `localStorage` on every change.
5. Restore saved notes from `localStorage` when the page loads.
6. Responsive card-style layout with a styled toolbar and Font Awesome icons.

### UX & Layout

- Fixed “Add note” button in the top-right corner of the viewport.
- Notes appear as cards that wrap across the page using a flexible layout.
- Each note includes:
  - A toolbar with Edit and Delete buttons.
  - A Markdown-rendered preview block.
  - A textarea for raw Markdown input, toggleable via the Edit button.

## High-Level Architecture and Flows

### Page Structure and Assets

- **HTML shell (`index.html`)**
  - Declares the root structure and the “Add note” button.
  - Loads:
    - `style.css` – local stylesheet for layout and theming.
    - Font Awesome CSS via CDN – provides toolbar icons.
    - Marked.js via CDN – Markdown-to-HTML rendering.
    - `script.js` – core client-side behavior, loaded with `defer`.

- **Styling (`style.css`)**
  - Imports Google Fonts (Poppins) via CDN.
  - Configures body as a flexible container for note cards.
  - Defines the fixed-position Add button and card styles (including toolbar and content areas).

### Runtime Flow

1. **Initial load**
   - Browser loads `index.html` and linked CSS/JS.
   - `script.js` runs after parsing, selecting the Add button via `document.getElementById("add")`.
   - Existing notes are loaded from `localStorage`:
     - Reads the `"notes"` key.
     - Parses it as JSON (array of Markdown strings) if present.
     - For each saved string, calls `addNewNote(savedText)` to reconstruct the note card.

2. **Creating a new note**
   - User clicks the Add Note button.
   - `addNewNote()` is called with no arguments, creating an empty note card.
   - The note is appended to `document.body`.
   - A textarea is shown (empty), and the rendered preview is initially hidden.

3. **Editing and live preview**
   - Each note has:
     - `.main` div – contains rendered HTML from `marked(markdownText)`.
     - `textarea` – contains raw Markdown.
   - Editing behavior:
     - Clicking the Edit button toggles `.hidden` on `.main` and `textarea`, switching between preview and edit modes.
     - On every `input` event in the textarea:
       - The new value is rendered using `marked(value)` into `.main`.
       - `updateLS()` is called to persist all notes.

4. **Deleting a note**
   - Clicking the Delete button removes the note’s DOM node.
   - `updateLS()` is called afterward to persist the updated set of notes (excluding the deleted one).

5. **Persistence with `localStorage`**
   - `updateLS()`:
     - Queries all `<textarea>` elements on the page.
     - Builds an array of their `.value` contents (one entry per note).
     - Serializes the array with `JSON.stringify` and saves it as the `"notes"` key in `localStorage`.
   - On the next load, the array is read back and used to recreate notes via `addNewNote(text)`.

### Use of External Assets

- **Marked.js (CDN)**
  - Loaded from `cdnjs` via `<script src="https://cdnjs.cloudflare.com/ajax/libs/marked/1.1.1/marked.min.js"></script>`.
  - Exposes a global `marked` function used in `script.js` to convert Markdown into HTML.

- **Font Awesome (CDN)**
  - Loaded from `cdnjs` via `<link rel="stylesheet" href="https://cdnjs.cloudflare.com/.../font-awesome/5.14.0/css/all.min.css">`.
  - Provides icons for the Add, Edit, and Delete buttons (e.g., `fas fa-plus`, `fas fa-edit`, `fas fa-trash-alt`).

- **Google Fonts (CDN)**
  - Imported in `style.css` with `@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@200;400;600&display=swap");`.
  - Used as the primary font for body and notes.

### Architecture Diagram (Described)

At a high level, the architecture can be visualized as:

- **Browser UI (HTML + CSS)**
  - Contains the Add button and note containers.
- **Client Logic (`script.js`)**
  - Handles events (click Add, click Edit/Delete, textarea input).
  - Manages DOM for notes and integrates with Marked.js.
  - Reads from and writes to `localStorage`.
- **Local Storage**
  - Persisted `"notes"` key (JSON array of Markdown strings).
- **CDN Dependencies**
  - Marked.js (Markdown rendering).
  - Font Awesome (icons).
  - Google Fonts (typography).

Data flows from the user’s input (textarea) into the client logic, which updates the DOM (preview) and `localStorage`. On reload, stored data flows back from `localStorage` into the recreated DOM via `addNewNote`.

## Assumptions and Limitations

### Assumptions

- The environment is a modern browser that supports:
  - `localStorage`.
  - ES5+ JavaScript features used in `script.js`.
  - Standard DOM APIs used for querying and event handling.
- Static hosting (e.g., Netlify) is used to serve the files; no backend or server-side processing is involved.
- The Marked.js version served by the CDN is compatible with the usage in `script.js` (global `marked` function).
- Icons and fonts are available via active CDN endpoints and a working internet connection.

### Limitations

- **No user accounts or cross-device sync**
  - Notes are scoped to a single browser and device. Clearing browser storage or switching devices loses the notes.
- **No note titles or metadata**
  - Each note is stored as plain Markdown text without titles, timestamps, or tags.
- **No ordering or search**
  - Notes are restored in the order stored in the array but cannot be reordered, filtered, or searched via the UI.
- **No conflict resolution**
  - There is no multi-tab or multi-user conflict handling; the latest persisted state simply overwrites prior values in `localStorage`.
- **Basic security**
  - Because this is static and local, there is no authentication, authorization, or server-side validation.

## Maintenance Notes

### Code Organization

- `index.html` – basic page skeleton, external asset loading, and the Add button.
- `style.css` – layout, typography, colors, and note card styling.
- `script.js` – all client-side logic, including:
  - Initial load and restoring from `localStorage`.
  - `addNewNote(text = "")` for creating and wiring up note cards.
  - `updateLS()` for persisting the current set of notes.

### Adding or Changing Features

- **New fields per note** (e.g., titles):
  - Extend the DOM template in `addNewNote` to include new elements.
  - Update `updateLS()` to store richer objects (e.g., array of `{text, title}`) instead of just strings.
  - Ensure backward compatibility when reading existing string-based notes.

- **Validation or sanitization**
  - If needed, configure Marked.js options (e.g., sanitization) before calling `marked`.
  - Consider adding simple content checks in the textarea `input` handler.

- **Theming or layout changes**
  - Most theming is centralized in `style.css`. Adjust background colors, note sizes, and typography there.
  - Maintain the `.hidden` class semantics used by the JS toggling logic.

### Operational Considerations

- Keep CDN URLs for Marked.js and Font Awesome up to date to avoid broken dependencies.
- For offline or more controlled deployments, consider self-hosting these assets.
- When changing Marked.js versions, verify that the `marked` API signature used by `script.js` is still supported.

## Security Considerations

- Markdown content is rendered into HTML using Marked.js, which may introduce XSS risks if unsafe HTML is allowed.
- The current code calls `marked(text)` without additional sanitization logic in `script.js`.
- For production-grade use, configure Marked.js with appropriate options or integrate additional sanitization to strip or escape unsafe HTML before inserting into the DOM.
