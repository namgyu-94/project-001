# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**sky-project-001** — Microsoft Dynamics 365 Business Central AL extension.

## Setup

1. Install [VS Code](https://code.visualstudio.com/) with the [AL Language extension](https://marketplace.visualstudio.com/items?itemName=ms-dynamics-smb.al).
2. Place a `launch.json` in `.vscode/` pointing to the target Business Central server (sandbox or on-prem).
3. Run **AL: Download Symbols** (Ctrl+Shift+P) to pull dependent `.app` symbols into `.alpackages/`.

## Build & Publish

| Action | VS Code Command |
|--------|----------------|
| Package (compile) | `AL: Package` → produces `*.app` |
| Publish to BC | `F5` or `AL: Publish without Debugging` |
| Download symbols | `AL: Download Symbols` |

## Project Structure

AL extensions follow a standard layout:

```
app.json          ← extension manifest (ID, version, dependencies)
.vscode/
  launch.json     ← server connection (gitignored — created per developer)
src/
  Tables/         ← table and table extension objects
  Pages/          ← page and page extension objects
  Codeunits/      ← business logic
  Reports/        ← report objects
  Enums/          ← enum and enum extension objects
```

`app.json` defines the extension `id` (GUID), `name`, `publisher`, `version`, `application` (minimum BC version), and `dependencies`.

## AL Language Patterns

- **Object declaration**: `table 50100 "My Table" { ... }`, `page 50100 "My Page" { ... }`, `codeunit 50100 "My Codeunit" { ... }`.
- **Table extensions**: `tableextension 50100 "Customer Ext" extends Customer { ... }` — use to add fields to base tables.
- **Page extensions**: `pageextension 50100 "Customer List Ext" extends "Customer List" { ... }`.
- **Procedures**: `procedure MyProc(param: Text): Integer` — `local` keyword restricts scope to the object.
- **Records**: `Rec.FindFirst()`, `Rec.FindSet()`, `Rec.Insert(true)`, `Rec.Modify(true)`, `Rec.Delete(true)` — the boolean triggers `OnInsert`/`OnModify`/`OnDelete` triggers.
- **Error handling**: `Error('message %1', value)` aborts with rollback; `Message()` is informational only.
- **Naming**: object numbers in the 50000–99999 range are for per-tenant extensions.
