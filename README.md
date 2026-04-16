# Claude Skills

A collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills.

## Skills

### gws-docs-create-from-markdown

Create a new Google Doc from a markdown file. The skill validates the file, checks for existing linked docs, optionally searches Google Drive for docs with the same title, and creates a new Google Doc. The Google Doc ID is stored in the markdown file as an HTML comment (`<!-- gdoc_id: <id> -->`) for use by the other skills.

**Install:** `npx skills add simonwahlgren/claude-skills --skill gws-docs-create-from-markdown`

**Usage:** `/gws-docs-create-from-markdown [title] [path-to-markdown-file]`

### gws-docs-update-from-markdown

Update an existing Google Doc by overwriting it with the contents of a local markdown file. Extracts the linked `gdoc_id` from the file and pushes the current markdown content to the doc. Warns that the update will unanchor existing comments.

**Install:** `npx skills add simonwahlgren/claude-skills --skill gws-docs-update-from-markdown`

**Usage:** `/gws-docs-update-from-markdown [path-to-markdown-file]`

### gws-docs-pull-comments

Pull unresolved comments from a Google Doc linked to a markdown file. Displays each comment with its author, creation date, quoted text, content, and any replies. Useful for reviewing feedback left on the Google Doc and addressing it in the local markdown file.

**Install:** `npx skills add simonwahlgren/claude-skills --skill gws-docs-pull-comments`

**Usage:** `/gws-docs-pull-comments [path-to-markdown-file]`
