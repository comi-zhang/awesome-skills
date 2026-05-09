---
name: lark-rich-doc-builder
description: Create and edit Feishu/Lark rich native documents and knowledge-base page systems with lark-cli. Use when the user asks Codex to create or improve a Feishu/Lark document, wiki page, parent-child document tree, rich text page, product spec, report, knowledge base, SOP, project hub, or documentation site that should use native Feishu features such as todos, callouts, tables, grids/columns, web bookmarks, URL previews, buttons, reminders, code blocks, images, attachments, sheets, whiteboards, Mermaid/PlantUML diagrams, child pages, or Wiki node hierarchy rather than plain Markdown.
---

# Lark Rich Doc Builder

## Overview

Use this skill to build Feishu/Lark documents as rich editable cloud pages, not as plain Markdown imports. The default output should combine native DocxXML blocks, optional Wiki parent-child structure, and visual/resource blocks when useful.

## Read First

Before touching live Feishu resources, read the relevant installed Lark skills:

- Always read `lark-shared` for authentication, identity, permission, and safety rules.
- Always read `lark-doc` and its XML/update references for `docs +create`, `docs +update`, and DocxXML syntax.
- Read `lark-wiki` when creating a knowledge-base page, parent page, child page, or shortcut node.
- Read `lark-drive` when inserting uploaded files, importing local files, searching resources, managing permissions, or adding comments.
- Read `lark-whiteboard` when generating or updating whiteboards.
- Read `lark-sheets` / `lark-base` after a document embeds or references sheets/base resources and the user wants their internal data edited.

Run health checks before write-heavy sessions:

```powershell
$env:SystemRoot='C:\Windows'
$env:windir='C:\Windows'
lark-cli doctor
lark-cli auth status
npx -y @larksuite/whiteboard-cli@^0.2.10 -v
```

The `SystemRoot` / `windir` assignments are important on Windows when DNS or Winsock provider lookup fails.

## Creation Decision Tree

- **Single rich document**: use `lark-cli docs +create --api-version v2 --doc-format xml`, then append/refine in sections.
- **Knowledge-base page or child pages**: use `lark-cli wiki +node-create` for the parent and child nodes; then edit each returned `obj_token`/URL with `docs +update`.
- **Existing doc improvement**: fetch with `docs +fetch --api-version v2 --detail with-ids`, then use targeted `block_insert_after`, `block_replace`, `str_replace`, or `append`; avoid `overwrite` unless the user explicitly wants a full rebuild.
- **Plain local file import**: if the user specifically wants to import `.md`, `.docx`, `.xlsx`, `.csv`, or `.base`, switch to `lark-drive +import` rather than recreating content manually.

## Rich Block Selection

Prefer DocxXML. Use native blocks deliberately:

| Need | Use |
|---|---|
| Checklist, action items, acceptance criteria | `<checkbox done="false">...</checkbox>` |
| Web page / external resource | `<bookmark name="..." href="..."></bookmark>` or `<a type="url-preview" href="...">...</a>` |
| Dense comparison or structured facts | `<table>` with `<thead>`, `<tbody>`, optional `background-color` |
| Side-by-side explanation | `<grid><column width-ratio="...">...</column></grid>` |
| Notes, warnings, conclusions | `<callout emoji="..." background-color="..." border-color="...">...</callout>` |
| Commands or examples | `<pre lang="..."><code>...</code></pre>` and inline `<code>` |
| Important links/actions | `<button action="OpenLink" src="...">...</button>` |
| Dates or reminders | `<time expire-time="..." notify-time="..." should-notify="false">...</time>` |
| Diagrams, maps, processes | `<whiteboard type="blank"></whiteboard>` then update via `lark-whiteboard` |
| Images / attachments | `docs +media-insert`, `<img href="..."/>`, or `<source name="..."/>` |
| Existing docs / users | `<cite type="doc" doc-id="..."></cite>` / `<cite type="user" user-id="..."></cite>` |
| Embedded spreadsheet | `<sheet type="blank"></sheet>` or copy an existing `<sheet token="..." sheet-id="..."></sheet>` |

Keep XML tags unescaped; only escape text content (`&`, `<`, `>`). Save large XML as a relative file such as `@./rich-doc-template.xml`; `lark-cli` rejects absolute `@file` paths.

## Recommended Workflow

### 1. Plan The Page System

Decide whether the deliverable is:

- one document;
- a Wiki parent page with child pages;
- a plain docx hub with linked docx children.

For maximum Feishu knowledge-base behavior, default to Wiki parent-child nodes. Use ordinary doc links only when the user does not want Wiki or no target Wiki space is available.

### 2. Create Parent And Children

For a single doc:

```powershell
lark-cli docs +create --api-version v2 --doc-format xml --as user `
  --content '<title>Title</title>'
```

For a Wiki parent page and child pages:

```powershell
lark-cli wiki +node-create --space-id my_library --title "Parent Title" --as user
lark-cli wiki +node-create --parent-node-token <parent_node_token> --title "Child Title" --as user
```

Capture `node_token`, `obj_token`, `obj_type`, `title`, and URL-like identifiers from each result. Edit `obj_token` documents with `docs +update`.

### 3. Write Rich XML In Passes

Start with a skeleton: title, opening callout, table of contents/child links, and major headings. Then append richer sections with `docs +update --command append` or targeted `block_insert_after`.

```powershell
lark-cli docs +update --api-version v2 --doc "<doc-url-or-token>" `
  --command append --doc-format xml --content "@./rich-doc-template.xml" --as user
```

Use `assets/rich-doc-template.xml` as the baseline. Replace placeholder content before running.

### 4. Add Visual And Resource Blocks

When adding whiteboards:

1. Insert `<whiteboard type="blank"></whiteboard>` in XML.
2. Read `data.document.new_blocks[]` and extract the `block_token` where `block_type == "whiteboard"`.
3. Render the local diagram preview.
4. Dry-run the whiteboard update.
5. Write only after dry-run is clean or the user confirms overwrite.

```powershell
npx -y @larksuite/whiteboard-cli@^0.2.10 -i .\assets\capability-map.mmd -o .\capability-map.png
cmd.exe /c "npx -y @larksuite/whiteboard-cli@^0.2.10 -i .\assets\capability-map.mmd --to openapi --format json | lark-cli whiteboard +update --whiteboard-token <board-token> --source - --input_format raw --idempotent-token <unique-10-plus-chars> --overwrite --dry-run --as user"
cmd.exe /c "npx -y @larksuite/whiteboard-cli@^0.2.10 -i .\assets\capability-map.mmd --to openapi --format json | lark-cli whiteboard +update --whiteboard-token <board-token> --source - --input_format raw --idempotent-token <unique-10-plus-chars> --overwrite --as user"
```

Prefer `cmd.exe /c` for the pipeline on Windows; PowerShell can fail to feed `--source -`.

### 5. Verify

Fetch each changed doc and, when whiteboards are used, export a preview:

```powershell
lark-cli docs +fetch --api-version v2 --doc "<doc-url-or-token>" --detail with-ids --as user
lark-cli whiteboard +query --whiteboard-token <board-token> --output_as image --output .\board-preview.png --as user
```

Return the parent Feishu/Wiki link first, then list child page links if created.

## Quality Bar

- Use at least three rich block families for non-trivial pages, for example callout + table + checklist + bookmark.
- Avoid long runs of plain `<p>` blocks; convert dense prose into tables, grids, checkboxes, or callouts.
- For knowledge bases, include a parent hub page that links or cites the child pages and explains navigation.
- Preserve existing resource tokens when editing existing documents; do not replace image, sheet, whiteboard, or file blocks with plain text.
- Use targeted edits over full overwrite to avoid losing comments, images, permissions context, and embedded resources.

## Safety Rules

- Do not print app secrets, access tokens, or config contents.
- Use `--as user` for user-owned documents unless the user explicitly asks for bot-owned resources.
- Use `--dry-run` before overwriting existing whiteboards or creating complicated Wiki node structures.
- Do not delete Wiki spaces, nodes, Drive files, or document blocks unless the user explicitly asks and confirms high-risk operations.
