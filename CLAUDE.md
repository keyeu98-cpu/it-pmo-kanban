# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A demo/training Kanban board for a fictitious bank's IT PMO. The whole app is one file, `index.html`, with an inline `<style>` and `<script>`. Don't use a real bank's logo or trademarks, and don't imitate an official bank system. The wordmark is plain text "IT PMO" and the palette is corporate blue.

## Hard constraints (from the original spec — keep them)

- Vanilla HTML/CSS/JS only. No frameworks, libraries, build step, npm, CDN scripts, web fonts or image files. Icons are Unicode glyphs or inline SVG, and fonts come from the system font stack.
- It must keep working when opened straight from disk (`file://`), with no server.
- **No persistence.** No localStorage, sessionStorage, IndexedDB or cookies. A refresh resets the board to the seed data on purpose, and the footer note says so.
- The only network call is the FormSubmit AJAX endpoint (`notifyNewTask`).
- No `alert()`/`confirm()` (deleting uses an inline "Delete? Yes / No" toggle) and no `!important`.

## Running and checking

- Run: `open index.html`, or `open -a Safari index.html`.
- There are no tests or linter. To syntax-check the script without Node, extract it and parse it with JavaScriptCore:
  ```sh
  sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/app.js
  osascript -l JavaScript -e 'ObjC.import("Foundation"); var s=$.NSString.stringWithContentsOfFileEncodingError("/tmp/app.js",4,null).js; try{new Function(s);"OK"}catch(e){"ERR "+e}'
  ```
- Constraint check: `grep -nE 'localStorage|sessionStorage|indexedDB|document\.cookie|https?://|confirm\(|alert\(|!important' index.html`. The only expected hit is the FormSubmit URL.

## Architecture

- **Config** sits at the top of the script: `FORMSUBMIT_ENDPOINT` (the one place the email address goes), `TASK_ID_PREFIX`, `NOTIFY_SUBJECT_TAG`. An HTML comment above the script explains FormSubmit's one-time activation: the first submission sends a confirmation email, and nothing is delivered until its link is clicked.
- **One state object** is the source of truth: `state = { tasks, filters, nextId, pendingDeleteId, openMoveId, focusAfterRender }`. Task IDs are `TASK_ID_PREFIX` plus the zero-padded `nextId`. The 8 seed tasks use IDs 0001–0008, and their due dates are relative to today, so some are always overdue.
- **Render from state, nowhere else.** Each mutation (`addTask`, `moveTask`, `deleteTask`) updates `state` and then calls `renderBoard()`, which rebuilds every column's HTML from `applyFilters(state.tasks)` and updates the count badges and `renderSummary()`. `renderCard()` returns an HTML string. Every user-supplied value must go through `escapeHtml()`. Don't change card DOM outside `renderBoard()`.
- **UI-only card state** (the open Move menu, the pending delete confirm) lives in `state` and appears when the cards are rendered, not through DOM toggling.
- **Focus restoration:** re-rendering destroys the focused element, so handlers set `state.focusAfterRender` (`{id, action}` or `{list: status}`) and `restoreFocus()` puts focus back after rendering. Keep this working for keyboard users whenever you add card actions.
- **Events are delegated:** one click and keydown handler on `#board` dispatches on `data-action` (`delete`, `delete-yes`, `delete-no`, `move-toggle`, `move-to`) and reads the card's `data-id`. Drag and drop uses the native HTML5 API: delegated `dragstart`/`dragend` on the board, plus per-column `dragenter`/`dragover`/`dragleave`/`drop`, which toggle `.drop-target`.
- **Add Task flow** (a `<dialog>` modal): `validateForm()` puts inline errors and `aria-invalid` on fields. If the form is valid, the card is added right away (optimistic), the form resets and a success toast shows. Then `notifyNewTask()` runs with the submit button disabled and showing "Sending…". Any failure, including FormSubmit's HTTP 200 with `success: "false"`, shows only a warning toast and never removes the card.
- **Styling:** CSS custom properties on `:root` for the palette, priority colours and spacing scale (`--space-1..6`). Priority shows as the card's left-border colour and as a text pill, so colour is never the only signal. Columns stack below 768px.
