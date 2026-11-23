# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

"Daily Reminder Delights": a to-do tracker in a single `index.html` (inline CSS and JS). The user wants plain HTML/CSS/JS with **no frameworks, no build step, no dependencies**. There are no tests, linter or package manager. Open the file in a browser to run it. The folder is not a git repository.

## Files

- `index.html`: the whole app. This is the only source file you edit.
- `mockup.html`: a **generated snapshot**, not hand-maintained source. It shows two copies of the app inside phone frames (iPhone 393×852, Galaxy S25 360×780), each in an `<iframe srcdoc>` seeded with demo tasks. It does not update when `index.html` changes.

## Architecture (index.html)

- State is one array `todos` of `{text, done, priority, category, due}`. `priority` is `high|medium|low`, `category` is one of `CATS` (Home, Work, Others), and `due` is a `YYYY-MM-DD` string or `''`.
- Every change calls `save()` and then `render()`. `render()` rebuilds the whole list and the "due tomorrow" banner (`renderAlert()`). Don't patch the DOM piecemeal.
- Persistence is `localStorage` under the keys `todos` and `theme`. All access is wrapped in try/catch, because storage throws in some contexts such as `data:` URLs and private windows. Keep it that way.
- The DOM is built with `createElement` and `textContent`, never `innerHTML`, so user text can't inject markup. Keep it that way.
- Sort order: priority (high first), then due date (earliest first, none last), then insertion order.
- Dates are compared as strings. `today()` and `tomorrow()` return local `YYYY-MM-DD`, and overdue or due-tomorrow labelling depends on that.
- Theming uses CSS variables on `:root` and `:root[data-theme=dark]`. With no saved choice it follows `prefers-color-scheme`. `applyTheme()` handles the Light/Dark selector.
- Edit mode is inline: the pencil button swaps the row into fields for text, category, date and priority, with Save and Cancel.
- Phone layout lives in a `@media (max-width:480px)` block under `/* Phones */`.
- Scope: the user asked for these features one at a time (add, delete, tick/untick, edit, priority, category, due dates, due-tomorrow alert, dark/light mode). Don't add features they haven't asked for.

## Regenerating mockup.html

After any change to `index.html`, `mockup.html` must be rebuilt. Its generator was a throwaway PowerShell script kept in the session scratchpad (`build-mockup.ps1` plus `mockup-template.html`), which may no longer exist. The method:

1. Take `index.html` and replace its final `render();</script>` with a seed block. The seed empties `todos` and pushes about four sample tasks, then sets `save = function(){}` so the demo never writes to storage. It also sets `document.body.style.paddingTop='40px'` for the notch and calls `render()`.
2. HTML-escape the result (`&` → `&amp;`, `"` → `&quot;`) and place it in both `srcdoc="…"` attributes of the page template. The template holds the h1, an h2 plus phone frame plus caption for each device, and the frame CSS.

## Testing and previewing

There is no test suite. Verify by opening `index.html` in the built-in browser and driving it. Previewing via `data:` URLs makes `localStorage` throw, so don't rely on persistence there. For device views, use the browser pane's `resize_window` (mobile preset) and reset it to `desktop` afterwards.
