
# RedM AI Knowledge Base

This repository is organized for AI retrieval first:

- one topic per file
- small files with predictable paths
- consistent YAML front matter
- separate docs, datasets, and rules

## What is here

- `docs/` contains explanation pages and implementation guidance
- `ai/` contains operating rules for coding agents
- `datasets/` contains structured source inventories and reference records
- `templates/` contains reusable page and resource templates

## How to use it

1. All you have to do is just add this link to your project's context
 ```markdown
 https://forzayt.github.io/RedM-Knowledge-Base
 ```
2. Start with `docs/getting-started/overview.md`
3. Read `ai/optimization-rules.md` and `ai/do-not-do.md`
4. Use `datasets/sources.json` for source discovery and filtering
5. Drill into framework, native, and tooling pages as needed

## Design goals

- predictable filenames for indexing
- shallow folder depth where possible
- machine-readable metadata on every page
- documentation and reference data kept separate

## Curated source groups

- Lua: official reference manual and Programming in Lua
- Cfx: resource manifests and RedM scripting docs
- Frameworks: VORP, RSG, RedEM, and Native Scripts ns-lib
- Natives: alloc8or, femga, nativewrappers, VORP RDR3 natives
- Tooling: OpenIV, Lenny's Mod Loader, and CodeWalker

## Next step

If you want this expanded, the next high-value pass is to add:

- per-native pages under `docs/natives/`
- per-framework subpages under `docs/frameworks/`
- JSON datasets for peds, horses, weapons, items, prompts, blips, and hashes
