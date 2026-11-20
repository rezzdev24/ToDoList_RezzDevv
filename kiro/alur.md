# Implementation Plan: Todo List Dashboard

## Overview

Build a zero-dependency, client-side dashboard delivered as three files (`index.html`, `css/style.css`, `js/app.js`). The implementation follows a scaffold-first order: static structure and theming are wired before any feature module, so every subsequent task can reference a working page and a live storage helper.

## Tasks

- [x] 1. Scaffold project files and IIFE skeleton
  - Create `index.html` at the project root with the full semantic HTML structure: `<header id="header">` containing `#greeting-widget` and `#theme-toggle`; `<main id="dashboard-grid">` containing `#timer-widget`, `#tasks-widget`, and `#links-widget`; link tags for `css/style.css` and `js/app.js`.
  - Create `css/style.css` with a CSS reset (box-sizing, margin/padding zeroed) and empty rule-sets for each widget section.
  - Create `js/app.js` as a single IIFE that calls `document.addEventListener('DOMContentLoaded', App.init)`. Define the top-level `App` object and six stub module objects — `Storage`, `Theme`, `Greeting`, `Timer`, `Tasks`, `QuickLinks` — each with an empty `init()` method. `App.init()` calls each module's `init()` in order.
  - Verify the page loads in a browser with no uncaught console errors.
  - _Requirements: 9.1, 9.2, 9.3, 9.4_

- [x] 2. Implement the Storage helper and CSS theming
  - [x] 2.1 Implement `Storage` module
    - Write `Storage.init()`: attempt a test write/read to `localStorage`; if a `SecurityError` is thrown, set `Storage.available = false`.
    - Implement `Storage.get(key)` → returns parsed JSON or the raw string; returns `null` on error or if unavailable.
    - Implement `Storage.set(key, value)` → serialises objects to JSON, writes to `localStorage`; no-op if unavailable.
    - Implement `Storage.remove(key)` → deletes the key; no-op if unavailable.
    - _Requirements: 9.5, 9.6_

  - [ ]* 2.2 Write unit tests for Storage module
    - Use a mock `localStorage` to simulate `SecurityError`; assert all methods return `null` / no-op when unavailable.
    - Assert `Storage.get` returns `null` for a key holding malformed JSON.
    - _Requirements: 9.5_

  - [x] 2.3 Implement CSS theme token system
    - Add `:root[data-theme="light"]` and `:root[data-theme="dark"]` rule-sets with the custom properties specified in the design (`--color-bg`, `--color-surface`, `--color-text`, `--color-text-muted`, `--color-accent`, `--color-border`).
    - Use `var()` references on all background, text, border, and accent declarations so switching `data-theme` on `<html>` applies the full theme change.
    - Add a `transition: background-color 0.3s, color 0.3s` rule on `body` and widget sections.
    - _Requirements: 8.1, 8.2, 8.5, 8.6_

  - [x] 2.4 Implement responsive grid layout
    - Style `#dashboard-grid` with `display: grid; grid-template-columns: 1fr` by default.
    - Add `@media (min-width: 768px)` overriding to `grid-template-columns: 1fr 1fr`.
    - _Requirements: 9.7, 9.8_

- [x] 3. Implement Light / Dark Mode toggle
  - [x] 3.1 Implement `Theme` module in `app.js`
    - Implement `Theme.apply(value)`: set `document.documentElement.setAttribute('data-theme', value)` and call `Storage.set('dashboard_theme', value)`.
    - Implement `Theme.init()`: call `Storage.get('dashboard_theme')`; if the stored value is `'light'` or `'dark'`, call `Theme.apply` with it; otherwise fall back to `matchMedia('(prefers-color-scheme: dark)').matches` to select `'dark'` or `'light'`, then call `Theme.apply`.
    - Add the `#theme-toggle` button to `index.html` inside `<header>`. On click, read the current `data-theme` attribute, flip it, and call `Theme.apply`.
    - Update the toggle button's label/icon to reflect the active theme after every `Theme.apply` call.
    - _Requirements: 8.1, 8.2, 8.3, 8.4_

  - [ ]* 3.2 Write unit tests for Theme module
    - Assert `Theme.init()` reads from storage before falling back to `matchMedia`.
    - Assert the toggle flip correctly alternates between `'light'` and `'dark'`.
    - _Requirements: 8.3, 8.4_

- [x] 4. Implement Greeting Widget
  - [x] 4.1 Implement clock, date, and time-of-day greeting
    - Add HTML elements to `#greeting-widget`: `#clock`, `#date-display`, `#greeting-text`, `#name-form` containing `#name-input` (`type="text" maxlength="50"`) and a submit button.
    - Implement `getGreetingPrefix(hour)`: return `"Good morning"` for 5–11, `"Good afternoon"` for 12–17, `"Good evening"` for 18–20, `"Good night"` for 21–23 and 0–4.
    - Implement `formatDate(date)`: return a locale string in the format "Weekday, DD Month YYYY".
    - Implement `tick()`: wrap `new Date()` in try/catch; on success update `#clock` with `HH:MM` and `#date-display`; on failure display the unavailability placeholder across all three elements.
    - In `Greeting.init()`, call `tick()` immediately and schedule `setInterval(tick, 1000)`.
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7_

  - [ ]* 4.2 Write property test for `getGreetingPrefix` (Property 1)
    - **Property 1: Time-of-day greeting covers all 24 hours without gaps or overlaps**
    - **Validates: Requirements 1.3, 1.4, 1.5, 1.6**
    - Use fast-check to generate integers in [0, 23]; assert the return value is one of the four allowed strings and is never empty or undefined.

  - [x] 4.3 Implement custom name persistence and display
    - On `#name-form` submit: trim the input value; if non-empty, call `Storage.set('dashboard_user_name', name)` and update `#greeting-text` to `getGreetingPrefix(hour) + ', ' + name`; if empty, call `Storage.remove('dashboard_user_name')` and display the prefix only.
    - In `Greeting.init()`, call `Storage.get('dashboard_user_name')`; if a value exists, pre-fill `#name-input` and update `#greeting-text`.
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8_

  - [ ]* 4.4 Write property test for name round-trip (Property 2)
    - **Property 2: Name greeting round-trip**
    - **Validates: Requirements 2.2, 2.3, 2.5**
    - Use fast-check to generate strings of 1–50 characters; simulate form submission; assert `#greeting-text` contains the name and `localStorage['dashboard_user_name']` equals the name.

- [ ] 5. Checkpoint — Greeting and theme complete
  - Ensure the page loads without console errors, the clock ticks, the greeting updates on name submit, and the theme toggle switches colors. Ask the user if questions arise.

- [x] 6. Implement Pomodoro Focus Timer
  - [x] 6.1 Implement core timer state and display
    - Add HTML elements to `#timer-widget`: `#timer-display`, `#timer-start`, `#timer-stop`, `#timer-reset` buttons, `#timer-banner` (hidden by default).
    - Implement `formatTimer(seconds)`: zero-pad minutes and seconds, return `"MM:SS"`.
    - Add internal state: `remainingSeconds`, `intervalId`, `running`, `originalTitle`.
    - Implement `Timer.render()`: write `formatTimer(remainingSeconds)` to `#timer-display`; enable/disable Start and Stop buttons according to `running`.
    - _Requirements: 3.1, 3.2_

  - [ ]* 6.2 Write property test for `formatTimer` (Property 3)
    - **Property 3: Timer display format is always valid MM:SS**
    - **Validates: Requirements 3.2, 3.9**
    - Use fast-check to generate integers in [0, 7200]; assert the output matches `^\d{2}:\d{2}$` and that encoded minutes and seconds are mathematically correct.

  - [x] 6.3 Implement Start, Stop, and Reset controls
    - `start()`: `setInterval(tick, 1000)`, set `running = true`, update tab title to `formatTimer(remainingSeconds) + ' ' + originalTitle`, call `Timer.render()`.
    - Each `tick()`: decrement `remainingSeconds`; call `Timer.render()`; update tab title; if `remainingSeconds === 0` call `complete()`.
    - `stop()`: `clearInterval(intervalId)`, set `running = false`, restore `document.title = originalTitle`, call `Timer.render()`.
    - `reset()`: call `stop()`, restore `remainingSeconds` to `pomodoroSeconds`, call `Timer.render()`.
    - `complete()`: call `stop()`, attempt `new Notification('Focus session complete!')`; if `Notification` is unsupported or permission is not `'granted'`, show `#timer-banner` with text "Session complete!" and hide it after 10 seconds (or on user dismiss).
    - In `Timer.init()`, save `originalTitle = document.title`, bind click handlers, call `Timer.render()`.
    - _Requirements: 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 3.10_

- [x] 7. Implement Configurable Pomodoro Duration
  - [x] 7.1 Implement duration input, validation, and persistence
    - Add `#duration-form` with `#duration-input` (`type="number" min="1" max="120"`) and a submit button to `#timer-widget`.
    - On form submit: parse the value as an integer; if outside [1, 120], display the message "Please enter a duration between 1 and 120 minutes." and return; otherwise update `pomodoroSeconds = value * 60`, call `Storage.set('dashboard_pomodoro_duration', value)`, and call `reset()`.
    - In `Timer.init()`, read `Storage.get('dashboard_pomodoro_duration')`; if the stored value is a valid integer in [1, 120], set `pomodoroSeconds` from it; otherwise default to `1500` (25 minutes).
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6_

  - [ ]* 7.2 Write property test for duration validation (Property 4)
    - **Property 4: Pomodoro duration validation accepts [1, 120] and rejects outside it**
    - **Validates: Requirements 4.1, 4.2, 4.3**
    - Use fast-check to generate integers both inside and outside [1, 120]; assert that valid inputs update the display to `D:00` and persist, while invalid inputs show the error message and retain the previous duration.

- [ ] 8. Checkpoint — Timer complete
  - Confirm countdown runs, tab title updates, Reset restores the display, the browser Notification fires (or the inline banner appears), and duration changes persist across reload. Ask the user if questions arise.

- [x] 9. Implement Task Management
  - [x] 9.1 Implement task data layer and persistence helpers
    - Add `#tasks-widget` HTML: `#task-form` with `#task-input` (`maxlength="200"`) and a submit button, `#task-error` span, `#task-list` (`<ul>`).
    - Implement `Tasks.persist()`: serialise the task array to JSON and call `Storage.set('dashboard_tasks', ...)`.
    - Implement `Tasks.load()`: call `Storage.get('dashboard_tasks')`; if the result is a valid array, assign it to the internal array; otherwise default to `[]`.
    - Implement the ID generator: `"t_" + Date.now() + "_" + counter++`.
    - _Requirements: 5.3, 5.4_

  - [ ]* 9.2 Write property test for task JSON round-trip (Property 7)
    - **Property 7: Task array JSON round-trip preserves all fields**
    - **Validates: Requirements 5.3, 5.4**
    - Use fast-check to generate arrays of Task objects; serialise and parse; assert deep equality.

  - [x] 9.3 Implement `addTask`, rendering, and complete toggle
    - `addTask(text)`: trim → reject if empty or > 200 chars (show `#task-error`) → pass to duplicate check (implemented in task 10.1) → push `{id, text: trimmed, done: false}` → persist → render.
    - `Tasks.renderList()`: clear `#task-list`; for each task append an `<li>` containing: a toggle button (complete/incomplete), a `<span>` with the task text (`text-decoration: line-through` if `done`), an edit button, and a delete button.
    - `toggleComplete(id)`: flip `done` → persist → re-render that `<li>`.
    - On `#task-form` submit, call `addTask`.
    - In `Tasks.init()`, call `Tasks.load()` then `Tasks.renderList()`.
    - _Requirements: 5.1, 5.2, 5.4, 5.5_

  - [x] 9.4 Implement inline edit and delete
    - `startEdit(id)`: replace the task `<span>` with an `<input>` pre-filled with the current text; show Confirm and Cancel buttons.
    - `confirmEdit(id, newText)`: trim → reject empty or > 200 chars (discard edit, restore label) → pass to duplicate check excluding self → update `text` → persist → re-render.
    - `cancelEdit(id)`: restore label without touching storage.
    - `deleteTask(id)`: splice from array → persist → remove `<li>`.
    - _Requirements: 5.6, 5.7, 5.8, 5.9, 5.10_

  - [ ]* 9.5 Write property test for task addition invariant (Property 5)
    - **Property 5: Task addition grows the list by exactly one and places the new task last**
    - **Validates: Requirements 5.1, 5.2, 5.4**
    - Use fast-check to generate a task list and a valid new description not already in the list; assert list length N+1, the new task is at index N, `done` is `false`, and `text` equals the trimmed input.

  - [ ]* 9.6 Write property test for whitespace and empty rejection (Property 6)
    - **Property 6: Whitespace-only and empty descriptions are rejected on add and edit**
    - **Validates: Requirements 5.1, 5.2, 5.8**
    - Use fast-check to generate whitespace-only and empty strings; assert `addTask` and `confirmEdit` both leave the list unchanged.

- [x] 10. Implement Duplicate Task Prevention
  - [x] 10.1 Implement `isDuplicate` and wire into add and edit
    - Implement `isDuplicate(tasks, text, excludeId)`: normalise by `text.trim().toLowerCase()`; return `true` if any task in the array (excluding the task with `excludeId` if supplied) has the same normalised text.
    - In `addTask`, call `isDuplicate(tasks, text)` after the length check; if true, set `#task-error` to the duplicate message and return.
    - In `confirmEdit`, call `isDuplicate(tasks, newText, id)` after the length check; if true, show the duplicate error and discard the edit.
    - Add an `input` event listener on `#task-input` to clear `#task-error` whenever the value no longer matches any existing task (and on the inline edit input field during an active edit).
    - _Requirements: 6.1, 6.2, 6.3, 6.4_

  - [ ]* 10.2 Write property test for duplicate prevention on add (Property 8)
    - **Property 8: Duplicate prevention on add (case-insensitive, whitespace-trimmed)**
    - **Validates: Requirements 6.1**
    - Use fast-check to generate a task list with ≥1 task and case/whitespace variants of an existing text; assert `addTask` rejects all and the list length is unchanged.

  - [ ]* 10.3 Write property test for duplicate prevention on edit (Property 9)
    - **Property 9: Duplicate prevention on edit excludes the task being edited**
    - **Validates: Requirements 6.3, 6.4**
    - Use fast-check to generate a task list with ≥2 tasks; assert editing to another task's text is rejected; assert editing to the task's own current text is accepted.

- [ ] 11. Checkpoint — Tasks complete
  - Confirm add, complete, edit, delete, and duplicate rejection all work. Verify persistence across a page reload. Ask the user if questions arise.

- [x] 12. Implement Quick Links
  - [x] 12.1 Implement link data layer, URL normalisation, and persistence helpers
    - Add `#links-widget` HTML: `#link-form` with `#link-label-input` and `#link-url-input`, a submit button, `#link-error` span, and `#link-list` (`<ul>`).
    - Implement `normaliseUrl(url)`: if the string does not start with `http://` or `https://`, prepend `https://`; return the string unchanged otherwise.
    - Implement `hasValidHost(url)`: attempt `new URL(url)`; return `true` if `hostname` is non-empty, `false` on parse error.
    - Implement `QuickLinks.persist()`: serialise to JSON and call `Storage.set('dashboard_quick_links', ...)`.
    - Implement `QuickLinks.load()`: read from storage; default to `[]` on absent or malformed JSON.
    - _Requirements: 7.4, 7.5, 7.7, 7.9_

  - [ ]* 12.2 Write property test for link JSON round-trip (Property 10)
    - **Property 10: Link array JSON round-trip preserves all fields**
    - **Validates: Requirements 7.4, 7.5**
    - Use fast-check to generate arrays of Link objects; serialise and parse; assert deep equality.

  - [ ]* 12.3 Write property test for URL normalisation idempotency (Property 11)
    - **Property 11: URL normalisation is idempotent and preserves valid prefixes**
    - **Validates: Requirements 7.7**
    - Use fast-check to generate arbitrary URL strings; assert `normaliseUrl(normaliseUrl(s)) === normaliseUrl(s)` and that strings already starting with `http://` or `https://` are returned unchanged.

  - [x] 12.4 Implement `addLink`, rendering, and delete
    - `addLink(label, url)`: trim both → validate label (1–50 chars), URL (1–2048 chars after trim), host validity after normalisation → check link count ≤ 50 → normalise URL → push `{id, label, url}` → persist → render.
    - `QuickLinks.renderList()`: clear `#link-list`; for each link append a `<li>` containing an `<a target="_blank" rel="noopener noreferrer">` button and a delete button.
    - `deleteLink(id)`: splice → persist → remove `<li>`.
    - Display field-specific error messages in `#link-error` for all validation failures.
    - In `QuickLinks.init()`, call `QuickLinks.load()` then `QuickLinks.renderList()`.
    - _Requirements: 7.1, 7.2, 7.3, 7.6, 7.8, 7.10_

- [-] 13. Responsive layout polish and final wiring
  - [x] 13.1 Apply widget-level styles and ensure no layout breakage
    - Style each widget section (`#timer-widget`, `#tasks-widget`, `#links-widget`) with surface color, padding, border-radius, and border using CSS custom properties.
    - Style the task list items, edit inputs, quick-link buttons, and all form controls to use the theme variables throughout.
    - Confirm the two-column grid at ≥ 768 px and single-column below 768 px with no overlapping or cropped content.
    - _Requirements: 9.4, 9.7, 9.8_

  - [ ] 13.2 Cross-browser smoke-test checklist (automated assertions)
    - Add a small inline test helper at the bottom of `app.js` (stripped before production) that asserts: page loads without uncaught errors, theme persists after `localStorage` manipulation, task array survives a serialise/parse cycle, timer display is valid MM:SS for boundary values, and duplicate task rejection fires for a known duplicate.
    - _Requirements: 9.4_

- [ ] 14. Final checkpoint — Ensure all tests pass
  - Run all property-based tests with `npx fast-check` (minimum 100 iterations per property). Confirm no uncaught console errors in Chrome, Firefox, Edge, and Safari. Confirm LocalStorage persistence for theme, tasks, links, name, and duration across a reload. Ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP delivery.
- Each task references specific requirements for traceability.
- Property tests use **fast-check** (run via `npx fast-check` — no install needed); each test must run a minimum of 100 iterations.
- The tag format for each property test is: `// Feature: todo-list-dashboard, Property N: <property_text>`
- Checkpoints ensure incremental validation at logical milestones.
- Because the app is a single IIFE, all property tests target the pure helper functions extracted from the modules (e.g., `getGreetingPrefix`, `formatTimer`, `isDuplicate`, `normaliseUrl`) — not DOM interactions — so they can run in Node without a browser.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1"] },
    { "id": 1, "tasks": ["2.1", "2.3", "2.4"] },
    { "id": 2, "tasks": ["2.2", "3.1"] },
    { "id": 3, "tasks": ["3.2", "4.1"] },
    { "id": 4, "tasks": ["4.2", "4.3"] },
    { "id": 5, "tasks": ["4.4", "6.1"] },
    { "id": 6, "tasks": ["6.2", "6.3"] },
    { "id": 7, "tasks": ["7.1"] },
    { "id": 8, "tasks": ["7.2", "9.1"] },
    { "id": 9, "tasks": ["9.2", "9.3"] },
    { "id": 10, "tasks": ["9.4", "9.5", "9.6"] },
    { "id": 11, "tasks": ["10.1"] },
    { "id": 12, "tasks": ["10.2", "10.3", "12.1"] },
    { "id": 13, "tasks": ["12.2", "12.3", "12.4"] },
    { "id": 14, "tasks": ["13.1"] },
    { "id": 15, "tasks": ["13.2"] }
  ]
}
```