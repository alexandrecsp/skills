# Graph

The `graph` branch of [`brain`](SKILL.md). The graph is [graph.html](graph.html), opened straight from this skill's folder. That folder's path changes with each plugin version, so this branch is how the user reaches the current page. The page keeps the brain folder it reads in the browser, under a key every copy of it shares, so the choice survives updates.

1. **Open it.** Build the page's URL from this skill's base directory: `file:///` followed by the absolute path to `graph.html`, with forward slashes. Open it with the OS's open command (`start "" "<url>"`, `open` or `xdg-open`). Done when the command ran, or failed.

2. **Hand over.** Give the user the URL and the brain's path, and say:
   - The page needs Chrome or Edge. When the default browser is another one, paste the URL into Chrome or Edge.
   - Click **Open brain** and pick the brain's folder, the one holding `BRAIN.md`.
   - After a browser restart, click **Reconnect**.
   - Notes cluster around their repository's hub. A node's shape gives its type, and dashed nodes are links to notes not written yet. Click a node to read its note.

   Done when the user has the URL.
