---
name: dashboard
description: Open the spec dashboard, one page showing the specs, milestones and tickets of every repository added to it.
disable-model-invocation: true
metadata:
  type: process
---

The dashboard is [dashboard.html](dashboard.html), opened straight from this skill's folder. That folder's path changes with each plugin version, so this skill is how the user reaches the current page. The page keeps its repo list in the browser, under a key that every copy of it shares, so the list survives updates.

## Process

1. **Open it.** Build the page's URL from this skill's base directory: `file:///` followed by the absolute path to `dashboard.html`, with forward slashes (`file:///C:/Users/<user>/.claude/plugins/cache/.../dashboard/dashboard.html`). Open it with the OS's open command (`start "" "<url>"`, `open` or `xdg-open`). Done when the command ran, or failed.

2. **Hand over.** Give the user the URL and say:
   - The page needs Chrome or Edge. When the default browser is another one, paste the URL into Chrome or Edge.
   - To list a repo, click **Add repo** and pick the repository's root folder, the one holding `.sdd`.
   - After a browser restart, each repo asks for permission again. Click **Reconnect** on its card.

   Done when the user has the URL.
