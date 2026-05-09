---
name: lark-visual-doc-builder
description: Create polished Feishu/Lark native documents with lark-cli by first creating an empty doc, then appending XML blocks, inserting blank whiteboards, and writing diagrams into those whiteboards with @larksuite/whiteboard-cli. Use when the user asks Codex to create or improve a Feishu/Lark document, especially when the document should contain rich native blocks such as callouts, grids, tables, code, whiteboards, Mermaid/PlantUML diagrams, capability maps, process diagrams, architecture diagrams, or other visual explanations rather than a plain Markdown import.
---

# Lark Visual Doc Builder

## Overview

Use this skill to create Feishu/Lark documents as real editable cloud docs, not as plain Markdown dumps. The preferred flow is: create an empty v2 document, append structured XML content, add one or more blank whiteboards, then render and write diagrams into those whiteboards.

## Prerequisites

- `lark-cli` must be installed and authenticated.
- `npx -y @larksuite/whiteboard-cli@^0.2.10 -v` must work before writing diagrams.
- On Windows, if DNS fails with Winsock/provider errors, set these for the current process before retrying:

```powershell
$env:SystemRoot='C:\Windows'
$env:windir='C:\Windows'
```

Check health before creating live content:

```powershell
lark-cli doctor
lark-cli auth status
```

If the standard Lark skills are installed, read them before acting:

- `lark-shared` for authentication, identity, permissions, and safety.
- `lark-doc` for v2 XML document creation/editing.
- `lark-whiteboard` for whiteboard query/update rules.

## Workflow

### 1. Draft Native XML

Prefer DocxXML over Markdown unless the user explicitly asks to import Markdown. Use rich blocks early:

- `<callout>` for the opening summary or warning.
- `<grid>` for side-by-side audience/use-case comparisons.
- `<table>` for capability matrices.
- `<whiteboard type="blank"></whiteboard>` as placeholders for diagrams.
- `<hr/>` between major sections.
- `<code>` for command names and flags.

Save substantial XML to a local file and pass it as `@./file.xml`; `lark-cli` rejects absolute `@file` paths.

See `assets/example-content.xml` for a reusable starting point.

### 2. Create An Empty Document

Create only the title first. This keeps creation reliable and makes the later edits auditable.

```powershell
lark-cli docs +create --api-version v2 `
  --content '<title>Document title</title>' `
  --doc-format xml --as user
```

Capture the returned document URL or `document_id`.

### 3. Append XML Content And Blank Whiteboards

Append the prepared XML file:

```powershell
lark-cli docs +update --api-version v2 `
  --doc "<doc-url-or-token>" `
  --command append `
  --content "@./content.xml" `
  --doc-format xml --as user
```

When the XML contains `<whiteboard type="blank"></whiteboard>`, the update response includes `data.document.new_blocks`. Extract the `block_token` where `block_type` is `whiteboard`; that is the `--whiteboard-token` for diagram writing.

### 4. Build And Preview The Diagram

For mind maps, sequence diagrams, class diagrams, pies, and simple flowcharts, Mermaid is usually enough. For complex layouts, prefer the `lark-whiteboard` DSL/SVG route if available.

Save Mermaid to `diagram.mmd`, then render a preview before writing to Feishu:

```powershell
npx -y @larksuite/whiteboard-cli@^0.2.10 `
  -i .\diagram.mmd `
  -o .\diagram.png
```

Inspect the PNG for empty output, text overflow, or incoherent layout. Revise before writing.

See `assets/example-capability-map.mmd` for a compact capability-map pattern.

### 5. Dry-Run And Write The Whiteboard

Always dry-run first. On PowerShell, prefer `cmd.exe /c` for the pipe because some `lark-cli --source -` reads can fail through a PowerShell pipeline.

```powershell
cmd.exe /c "npx -y @larksuite/whiteboard-cli@^0.2.10 -i .\diagram.mmd --to openapi --format json | lark-cli whiteboard +update --whiteboard-token <board-token> --source - --input_format raw --idempotent-token <10-plus-char-token> --overwrite --dry-run --as user"
```

If the dry-run says existing nodes will be deleted, ask the user before proceeding. For a newly-created blank whiteboard, proceed:

```powershell
cmd.exe /c "npx -y @larksuite/whiteboard-cli@^0.2.10 -i .\diagram.mmd --to openapi --format json | lark-cli whiteboard +update --whiteboard-token <board-token> --source - --input_format raw --idempotent-token <10-plus-char-token> --overwrite --as user"
```

Use a unique idempotent token such as `202605092350-map`.

### 6. Verify And Return The Feishu Link

Fetch the document and export a whiteboard preview if possible:

```powershell
lark-cli docs +fetch --api-version v2 --doc "<doc-url>" --detail with-ids --as user
lark-cli whiteboard +query --whiteboard-token <board-token> --output_as image --output .\board-preview.png --as user
```

Only finish after the document fetch succeeds and the whiteboard preview is non-empty. Return the Feishu document link as the primary result.

## Safety Rules

- Do not print app secrets, access tokens, or config file contents.
- Use `--as user` for user-owned docs unless the user explicitly wants bot-owned resources.
- Use `--dry-run` before overwriting any existing whiteboard content.
- Prefer incremental `append` / targeted edits over replacing a whole document unless the user asks for a full rewrite.
