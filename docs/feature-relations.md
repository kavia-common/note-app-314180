# Notes App 314180 – Features and Implementation Mapping

## Overview

This document maps the main product features of Notes App 314180 to their concrete implementation details: UI elements, storage interactions, and external libraries. It serves as a quick reference for developers modifying or extending the app.

## Features List

1. Create new notes via an Add Note button.
2. Edit notes in a textarea with live Markdown preview using Marked.js.
3. Delete notes individually from the UI and from browser storage.
4. Automatic persistence of all notes to browser `localStorage`.
5. Restore saved notes on page load in the same browser.
6. Responsive card-style layout with toolbar and icons.

## Feature-to-Implementation Matrix

### Legend

- **UI** – HTML elements and CSS classes involved.
- **Logic** – Functions and event handlers in `script.js`.
- **Storage** – How and when `localStorage` is read or written.
- **Libraries** – External dependencies involved.

### 1. Create New Notes

**Behavior:** Clicking the Add Note button creates and displays a new, empty note card.

- **UI**
  - HTML:
    - `index.html`:
      - `<button class="add" id="add"><i class="fas fa-plus"></i> Add note</button>`
  - CSS:
    - `.add` styles the fixed-position button in the top-right.
    - `.note` defines the note card container.
- **Logic**
  - Variable:
    - `const addBtn = document.getElementById("add");`
  - Event handler:
    - `addBtn.addEventListener("click", () => { addNewNote(); });`
  - Function:
    - `function addNewNote(text = "") { ... }`
      - Creates a new `<div>` with class `note`.
      - Injects inner HTML template containing:
        - `.tools` toolbar with Edit/Delete buttons.
        - `.main` div for rendered preview.
        - `<textarea>` for Markdown input.
      - Appends the new note element to `document.body`.
- **Storage**
  - Creation itself does not immediately write to `localStorage`.
  - `localStorage` is updated the first time the user types in the new note (via textarea `input` and `updateLS()`).
- **Libraries**
  - Font Awesome:
    - `fas fa-plus` icon on the Add button.
  - No direct Marked.js usage at creation time (until text is rendered).

### 2. Edit Notes with Live Markdown Preview

**Behavior:** Users edit a note’s content in a textarea, and a preview area renders the Markdown to HTML in real time.

- **UI**
  - Template inside `addNewNote`:
    - `<div class="notes">`
      - `<div class="tools">`
        - `<button class="edit"><i class="fas fa-edit"></i></button>`
        - `<button class="delete"><i class="fas fa-trash-alt"></i></button>`
      - `</div>`
      - `<div class="main ${text ? "" : "hidden"}"></div>`
      - `<textarea class="${text ? "hidden" : ""}"></textarea>`
      - `</div>`
  - CSS:
    - `.note .main` – container for rendered HTML preview.
    - `.note textarea` – styled input area for Markdown.
    - `.note .hidden` – toggles display on and off.
- **Logic**
  - Within `addNewNote`:
    - Element references:
      - `const editBtn = note.querySelector(".edit");`
      - `const main = note.querySelector(".main");`
      - `const textArea = note.querySelector("textarea");`
    - Initial setup:
      - `textArea.value = text;`
      - `main.innerHTML = marked(text);`
    - Edit toggle:
      - `editBtn.addEventListener("click", () => {`
        - `main.classList.toggle("hidden");`
        - `textArea.classList.toggle("hidden");`
      - `});`
    - Live preview and persistence:
      - `textArea.addEventListener("input", (e) => {`
        - `const { value } = e.target;`
        - `main.innerHTML = marked(value);`
        - `updateLS();`
      - `});`
- **Storage**
  - Every `input` event triggers `updateLS()`, ensuring that any change is immediately persisted.
- **Libraries**
  - Marked.js:
    - Global `marked` function called as `marked(text)` and `marked(value)` to render Markdown into `.main`.

### 3. Delete Notes

**Behavior:** Clicking the Delete button removes a note from the page and updates `localStorage` to remove it from persistence.

- **UI**
  - Template in `addNewNote`:
    - `<button class="delete"><i class="fas fa-trash-alt"></i></button>`
  - CSS:
    - `.note .tools button` – base styling for toolbar buttons.
- **Logic**
  - Within `addNewNote`:
    - `const deleteBtn = note.querySelector(".delete");`
    - `deleteBtn.addEventListener("click", () => {`
      - `note.remove();`
      - `updateLS();`
    - `});`
- **Storage**
  - After a note is removed from the DOM, `updateLS()` reconstructs the notes array using all remaining `<textarea>` values, then writes it back to `localStorage`, effectively removing the deleted note’s content.
- **Libraries**
  - Font Awesome:
    - `fas fa-trash-alt` icon on the Delete button.

### 4. Automatic Persistence to localStorage

**Behavior:** Any change in any note’s content results in the whole notes set being saved to `localStorage`.

- **UI**
  - All `<textarea>` elements across all notes participate.
- **Logic**
  - Function `updateLS()`:
    - `const notesText = document.querySelectorAll("textarea");`
    - Builds:
      - `const notes = [];`
      - `notesText.forEach((note) => { notes.push(note.value); });`
    - Writes:
      - `localStorage.setItem("notes", JSON.stringify(notes));`
  - Trigger points:
    - Text changes:
      - `textArea.addEventListener("input", ...)` calls `updateLS()`.
    - Note deletions:
      - Delete handler calls `updateLS()` after `note.remove()`.
- **Storage**
  - Uses a single key:
    - Key: `"notes"`
    - Value: JSON array of strings (one per note, representing raw Markdown).
- **Libraries**
  - Native browser `localStorage` API, no external library wrapper.

### 5. Restore Notes on Page Load

**Behavior:** When the user revisits or reloads the app in the same browser, previously saved notes are reconstructed from `localStorage`.

- **UI**
  - The DOM is initially empty apart from the Add button; all notes are dynamically created.
- **Logic**
  - At the top of `script.js`:
    - `const notes = JSON.parse(localStorage.getItem("notes"));`
    - Conditional recreation:
      - `if (notes) {`
        - `notes.forEach((note) => {`
          - `addNewNote(note);`
        - `});`
      - `}`
  - Each entry loaded from `localStorage` is passed as `text` to `addNewNote(text)`, which:
    - Sets `textArea.value = text;`
    - Renders `main.innerHTML = marked(text);`
    - Shows `.main` and hides `textarea` by default when `text` is non-empty.
- **Storage**
  - Reads from `localStorage` only once during initial script execution (page load).
- **Libraries**
  - Marked.js is used during note reconstruction to produce the initial rendered HTML from stored Markdown.

### 6. Responsive Card-Style Layout and Icons

**Behavior:** Notes are displayed as cards that wrap across the viewport, with a consistent toolbar, icons, and a light theme.

- **UI**
  - `body`:
    - `display: flex;`
    - `flex-wrap: wrap;`
    - `padding-top: 3rem;`
    - `background-color: #7bdaf3;`
  - `.note`:
    - Fixed height/width for consistent card sizing.
    - `box-shadow` for elevation.
  - `.note .tools`:
    - `display: flex; justify-content: flex-end;`
    - Background color matching primary theme color.
  - `.add`:
    - `position: fixed; top: 1rem; right: 1rem;`
    - Theming with primary button color and white text.
- **Logic**
  - The layout is purely CSS-driven; JavaScript only appends `.note` containers to `document.body`.
- **Storage**
  - Not directly involved.
- **Libraries**
  - Font Awesome for icons:
    - `fas fa-plus`, `fas fa-edit`, `fas fa-trash-alt`.
  - Google Fonts (Poppins) for typography.

## High-Level Architecture / Flow (Developer-Focused)

### Core JS Functions

- `addNewNote(text = "")`
  - Creates a new note card element.
  - Initializes `textarea` and `.main` with the provided text.
  - Sets up event listeners for edit toggling, deletion, and live preview plus persistence.
  - Appends the completed note to `document.body`.

- `updateLS()`
  - Aggregates all note contents by reading `value` from each `<textarea>`.
  - Serializes the array and writes it to `localStorage` under the `"notes"` key.

### Data Flow Summary

1. **User input (textarea) → Live preview**
   - User types in `<textarea>`.
   - JS handler uses `marked(value)` to produce HTML.
   - HTML is injected into `.main`.

2. **User input (textarea) → localStorage**
   - Same `input` handler calls `updateLS()`.
   - `updateLS()` collects all textarea values into an array.
   - Array is serialized and stored in `localStorage`.

3. **localStorage → DOM reconstruction**
   - On load, `localStorage.getItem("notes")` returns the array of Markdown strings.
   - Each string is passed to `addNewNote`.
   - `addNewNote` rebuilds DOM nodes and preview content.

## Assumptions and Limitations (Feature Perspective)

- All notes are stored as plain Markdown strings without additional metadata.
- Deleting a note is permanent from the perspective of the app; there is no undo.
- The full state is derived solely from `localStorage` and the DOM at runtime.
- The app assumes a working Marked.js global and valid JSON in the `"notes"` key.

## Maintenance Notes

- When adding new note attributes (such as titles or tags), update:
  - The `addNewNote` DOM template.
  - The logic that reads initial state from `localStorage`.
  - The `updateLS()` implementation and any code that interprets the stored structure.
- If Marked.js is replaced or upgraded:
  - Ensure that `marked(text)` is still valid and that necessary security/sanitization options are configured.
- If you introduce new UI states (e.g., pinned notes, archived notes), ensure that:
  - Their state is captured in the persisted JSON structure.
  - They are reconstructed correctly on load.
