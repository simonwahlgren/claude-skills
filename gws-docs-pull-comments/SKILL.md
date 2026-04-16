---
name: gws-docs-pull-comments
description: Pull comments from a Google Doc linked to a markdown file using the gws CLI
disable-model-invocation: false
user-invocable: true
argument-hint: "[path-to-markdown-file]"
---

# Pull Comments from Google Doc

Fetch all unresolved comments from a Google Doc linked to a local markdown file using the `gws` CLI. Displays each comment with the quoted text it refers to, making it easy to address feedback locally.

## Inputs

This skill expects a **path to a markdown file** that has a `gdoc_id` stored as `<!-- gdoc_id: <id> -->`.

- If `$ARGUMENTS` contains a file path, use it directly.
- If no argument is provided, ask the user for the path.
- Only `.md` files are supported. If the user provides a non-markdown file, tell them and stop.

## Steps

1. **Validate the file** — confirm the path exists and ends with `.md`.

2. **Read the `gdoc_id`** — read the file and look for an HTML comment in the format `<!-- gdoc_id: <id> -->` (typically on the first line).
   - If no `gdoc_id` is found, tell the user: "This file has no linked Google Doc. Use `/gws-docs-create-from-markdown` to create one first." Then stop.

3. **Fetch comments** — run:
   ```bash
   gws drive comments list \
     --params '{"fileId": "<gdoc_id>", "fields": "comments(id,author(displayName),content,quotedFileContent,resolved,createdTime,replies(author(displayName),content,createdTime))", "pageSize": 100}' \
     --page-all
   ```

4. **Filter and display** — parse the JSON response and show only **unresolved** comments (where `resolved` is not `true`). For each comment, display:

   ```
   ---
   Comment by <author> (<date>):
   > Quoted text: "<quotedFileContent.value>"
   
   <comment content>
   
   Replies:
     - <reply author> (<date>): <reply content>
   ---
   ```

   If there are no unresolved comments, tell the user: "No unresolved comments found."

5. **Offer to help** — after displaying the comments, ask the user if they'd like help addressing any of them in the markdown file. If so, read the full markdown file, locate the relevant sections using `quotedFileContent`, and suggest or apply edits.
