# Workflow Notes

## Why Create Empty First

Creating a title-only document first separates resource creation from content mutation. It makes failures easier to recover from, and it allows later steps to return concrete block tokens for embedded resources such as whiteboards.

## Useful Checks

```powershell
lark-cli doctor
lark-cli docs +create --api-version v2 --help
lark-cli docs +update --api-version v2 --help
lark-cli whiteboard +update --help
npx -y @larksuite/whiteboard-cli@^0.2.10 -v
```

## Common Pitfalls

- `@file` paths must be relative to the current directory.
- Use `--api-version v2` for `docs +create`, `docs +fetch`, and `docs +update`.
- If `lookup mcp.feishu.cn` fails on Windows with provider errors, set `SystemRoot` and `windir` to `C:\Windows` in the process environment.
- PowerShell pipelines can fail to feed `--source -`; use `cmd.exe /c "producer | lark-cli ... --source -"` for whiteboard updates.
- Do not treat a Markdown import as “native document editing” when the user asked for rich Feishu blocks or visual elements.
