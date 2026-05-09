# Wiki Parent/Child Page Template

Use this structure when a user asks for a knowledge base, document system, handbook, project hub, or multiple child documents.

## Parent Page

- Title: concise hub name.
- Opening callout: purpose, audience, update cadence.
- Navigation table: child page title, owner, purpose, status.
- Decision or overview section: summarize the whole system.
- Resource section: bookmarks, related docs, sheets, or external links.

## Child Pages

Recommended child page types:

- Overview / background
- Requirements / scope
- Execution plan / SOP
- References / appendix
- Changelog / meeting notes

## Commands

```powershell
lark-cli wiki +node-create --space-id my_library --title "Parent Title" --as user
lark-cli wiki +node-create --parent-node-token <parent_node_token> --title "Child Title" --as user
lark-cli docs +update --api-version v2 --doc <obj_token-or-url> --command append --doc-format xml --content "@./child-content.xml" --as user
```

When Wiki is not desired or unavailable, create ordinary docx documents and add `<bookmark>` / `<cite type="doc">` references from the hub document.
