# Workflow Notes

## Native Rich Document Principles

- Prefer DocxXML for creation and edits. Markdown is acceptable only for explicit imports or user-provided Markdown.
- Create long documents in passes: title/skeleton first, then append sections, then refine block-level details.
- Use rich block types to reduce plain prose: checklist, callout, grid, table, bookmark, button, time, whiteboard, media.
- Fetch with `--detail with-ids` before precise edits so block-level operations can target stable IDs.

## Wiki Parent/Child Strategy

- Default to `wiki +node-create` for multi-page knowledge bases.
- Use `--space-id my_library` for the user's personal knowledge base unless the user gives a specific space or parent node.
- Use `--parent-node-token` to create child pages under a parent.
- Capture `node_token` for hierarchy operations and `obj_token` for document content edits.
- If Wiki is not requested or fails due to permissions, fall back to ordinary docx documents linked from a hub page.

## Windows And CLI Pitfalls

```powershell
$env:SystemRoot='C:\Windows'
$env:windir='C:\Windows'
```

Set these when DNS errors mention Winsock providers or `getaddrinfow`.

Other pitfalls:

- `@file` paths must be relative to the current working directory.
- `docs +create`, `docs +fetch`, and `docs +update` must include `--api-version v2`.
- PowerShell pipelines may not feed `lark-cli --source -`; use `cmd.exe /c "producer | lark-cli ... --source -"`.
- Dry-run whiteboard overwrites before executing.
- Do not overwrite whole documents unless the user asks for a full rebuild.
