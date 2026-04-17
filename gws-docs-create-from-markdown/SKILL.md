---
name: gws-docs-create-from-markdown
description: Create a new Google Doc from a markdown file using the gws CLI
disable-model-invocation: false
user-invocable: true
argument-hint: "[title] [path-to-markdown-file]"
---

# Create Google Doc from Markdown

Create a new Google Doc by uploading and converting a local markdown file using the `gws` CLI.

## Inputs

This skill expects two inputs: a **title** and a **path to a markdown file**.

- If `$ARGUMENTS` contains both (e.g. `"My RFC Title" ./path/to/file.md`), use them directly.
- If either is missing, ask the user for the missing value(s).
- Only `.md` files are supported. If the user provides a non-markdown file, tell them and stop.

## Steps

1. **Validate the file** — confirm the path exists and ends with `.md`. Read the file to check for an existing `gdoc_id`.

2. **Check for existing `gdoc_id`** — read the file and look for an HTML comment in the format `<!-- gdoc_id: <id> -->` (typically on the first line).
   - If `gdoc_id` is already set to a non-empty value, tell the user: "This file already has a Google Doc linked (gdoc_id: `<id>`). Use `/gws-docs-update-from-markdown` to update it instead." Then stop.

3. **Check for a previously linked doc** — search Google Drive for a doc with the exact same title:
   ```bash
   gws drive files list --params '{"q": "name = \"<title>\" and mimeType = \"application/vnd.google-apps.document\" and trashed = false", "fields": "files(id,name,createdTime)", "supportsAllDrives": true, "includeItemsFromAllDrives": true}'
   ```
   - If a matching doc is found, ask the user:
     > "Found an existing Google Doc with this title (id: `<id>`, created: `<date>`). Would you like to:
     > 1. **Re-link** — add this doc's ID to your file (no new doc created)
     > 2. **Create new** — create a fresh Google Doc (the existing one will remain in Drive)"
   - If the user chooses re-link, skip to step 6 (store the existing doc ID).
   - If the user chooses create new, continue to step 4.
   - If no matching doc is found, continue to step 4.

4. **Create the Google Doc** — run:
   ```bash
   gws drive files create \
     --json '{"name": "<title>", "mimeType": "application/vnd.google-apps.document"}' \
     --upload <path-to-markdown-file> \
     --upload-content-type text/markdown
   ```

5. **Extract the doc ID** — parse the JSON response and extract the `id` field.

6. **Set pageless mode** — convert the doc to pageless format:
   ```bash
   gws docs documents batchUpdate \
     --params '{"documentId": "<id>"}' \
     --json '{"requests": [{"updateDocumentStyle": {"documentStyle": {"documentFormat": {"documentMode": "PAGELESS"}}, "fields": "documentFormat"}}]}'
   ```

7. **Store the doc ID** — — edit the markdown file to add `<!-- gdoc_id: <id> -->` as the very first line of the file. This HTML comment is invisible in Google Docs (stripped during conversion) but readable locally by the skills.
   - If the file already has a `<!-- gdoc_id: ... -->` line, replace it.
   - If not, prepend it as the first line followed by a blank line.

7. **Report success** — show the user:
   - The Google Doc ID
   - The Google Doc URL: `https://docs.google.com/document/d/<id>/edit`
   - Confirm the `gdoc_id` was saved to the file.
