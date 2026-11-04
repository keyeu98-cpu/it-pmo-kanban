# IT PMO Kanban

A demo Kanban board for a fictitious bank's IT project management office (PMO). It's for training and demos only and isn't connected to any real bank's systems.

The whole app is one file, `index.html`, written in plain HTML, CSS and JavaScript. It has no dependencies, no build step and no external resources.

## Run it

Download `index.html` and double-click it. No server is needed.

To host it, turn on GitHub Pages for this repo (**Settings → Pages → Deploy from a branch → `main` / root**). The board will be at `https://<your-user>.github.io/<repo-name>/`.

## Features

- Four columns: Backlog, In Progress, Blocked and Done. Each has a live task count.
- Move cards by dragging them, or with the keyboard using the **Move ▸** button on each card.
- Each card's left edge is coloured by priority, and its priority is also written as text. Tasks past their due date get an **Overdue** badge.
- Deleting a card asks "Delete? Yes / No" on the card itself, with no pop-up dialog.
- Filter by project, assignee and priority.
- The header shows live totals for each status and for overdue tasks.
- The **Add Task** form checks each field and shows errors under it. It can also send an email notification through [FormSubmit](https://formsubmit.co).

## Data resets on refresh

Tasks are kept in memory only, so refreshing the page brings back the 8 sample tasks. This is deliberate for a demo.

## Email notifications (FormSubmit)

To get an email for each new task:

1. Near the top of the `<script>` block in `index.html`, change the email address in `FORMSUBMIT_ENDPOINT`:
   ```js
   const FORMSUBMIT_ENDPOINT = "https://formsubmit.co/ajax/YOUR_EMAIL@example.com";
   ```
2. Activate the address. The first task you add doesn't send a notification. FormSubmit emails that address a confirmation link instead, and notifications arrive only after you click it.

If you put a real email address in a public repo, anyone can read it in the source.

If a notification fails, the card still stays on the board and the page shows the warning "Card added locally — email notification failed".
