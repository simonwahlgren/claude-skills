---
name: gws-docs-update-from-markdown
description: Update an existing Google Doc by overwriting it with a local markdown file using the gws CLI
disable-model-invocation: false
user-invocable: true
argument-hint: "[path-to-markdown-file]"
---

# Update Google Doc from Markdown

Overwrite an existing Google Doc with the contents of a local markdown file using the `gws` CLI.

## Inputs

This skill expects a **path to a markdown file** that has a `gdoc_id` in its YAML frontmatter.

- If `$ARGUMENTS` contains a file path, use it directly.
- If no argument is provided, ask the user for the path.
- Only `.md` files are supported. If the user provides a non-markdown file, tell them and stop.

## Steps

1. **Validate the file** — confirm the path exists and ends with `.md`.

2. **Read the `gdoc_id`** — read the file and look for an HTML comment in the format `<!-- gdoc_id: <id> -->` (typically on the first line).
   - If no `gdoc_id` is found, tell the user: "This file has no linked Google Doc. Use `/gws-docs-create-from-markdown` to create one first." Then stop.

3. **Confirm with the user** — before proceeding, ask:
   > "This will overwrite the Google Doc (`<gdoc_id>`) with the contents of `<filename>`. Any comments on the doc will become unanchored and hidden from the UI (they can still be retrieved). Make sure you've pulled and addressed all comments before updating. Proceed?"

   Wait for the user to confirm. If they decline, stop.

4. **Update the Google Doc** — run:
   ```bash
   gws drive files update \
     --params '{"fileId": "<gdoc_id>"}' \
     --upload <path-to-markdown-file> \
     --upload-content-type text/markdown
   ```

5. **Report success** — show the user:
   - Confirmation that the doc was updated
   - The Google Doc URL: `https://docs.google.com/document/d/<gdoc_id>/edit`
