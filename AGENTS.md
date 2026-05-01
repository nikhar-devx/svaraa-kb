# Agent Context — Svaraa Knowledge Base

Knowledge base for **Svaraa Website** and **Svaraa POS**. All notes are flat `.md` files directly under `content/`. No subfolders. Organization is entirely via frontmatter tags.

---

## Tag Taxonomy

Every note MUST have at least 2 tags: a **type** tag and a **domain** tag.

### Level 1: Note Type (pick one)
- `#inbox` — Unprocessed, needs sorting
- `#architecture` — Code-related: modules, APIs, data models, sync flows, service APIs
- `#reference` — Everything else: URLs, config, processes, quick lookups

### Level 2: Domain (pick one)
- `#storefront` — Customer-facing website
- `#pos` — Point of Sale system
- `#backend` — Medusa backend / API layer
- `#design` — UI/UX
- `#ops` — DevOps, infrastructure

### Level 3: Topic (optional)
- `#auth` `#catalog` `#cart` `#checkout` `#payments`
- `#inventory` `#orders` `#seo` `#search` `#cms`

---

## Frontmatter

```yaml
---
title: Note Title
created: YYYY-MM-DD
domain: storefront | pos | backend | design | ops
tags:
  - type-tag
  - domain-tag
  - optional-topic-tag
---
```

---

## Writing Rules

1. **No definition-only notes.** If it's in public docs with no project context, don't write it.
2. **Every note answers:** "What does someone on this project need to know that isn't obvious from the code or external docs?"
3. **`#architecture` notes** cover: how a module works, data models, sync flows, service APIs, debugging.
4. **`#reference` notes** cover: URLs, config values, environment details, processes, quick lookups.

---

## Sort Inbox

Use `/sort-inbox`. It finds all notes tagged `#inbox`, determines the correct type and domain tags, rewrites frontmatter, and removes the `#inbox` tag.

Decision:
```
Note tagged #inbox
├── Code-related? (module, API, data model, sync flow, service) → #architecture
└── Everything else?                                            → #reference
```

---

## Safety

- Never delete notes. Tag with `#trash` and leave in place if junk.
- Preserve existing wiki links (`[[...]]`) when editing files.
