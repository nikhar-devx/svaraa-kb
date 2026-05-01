---
name: sort-inbox
description: Find all notes tagged #inbox in content/, classify as #architecture (code-related) or #reference (everything else), update frontmatter tags, and remove the #inbox tag.
---

# Sort Inbox

Process every `.md` file in `@../content/` that has `#inbox` in its tags.

## Workflow

1. **Discover** → Grep for `#inbox` in frontmatter across all `.md` files in `@../content/`
2. **Read** → Read each matched file
3. **Classify** → Apply the decision below
4. **Rewrite frontmatter** → Replace `#inbox` with correct type + domain tags
5. **Report** → File name and tags applied

## Classification

```
Note tagged #inbox
├── Code-related? (module internals, API routes, data models, sync flows,
│   service APIs, database schemas, event subscribers, debugging)
│   └── type tag: #architecture
│
└── Everything else? (URLs, config, environment, processes,
    meeting outcomes, quick lookups, SEO data)
    └── type tag: #reference
```

If genuinely unclear, keep `#inbox` and add a `> ⚠️ Needs review` callout at the top of the note body.

## Frontmatter After Sorting

```yaml
---
title: Note Title
created: YYYY-MM-DD
domain: storefront | pos | backend | design | ops
tags:
  - architecture | reference
  - <domain>
  - <optional topic>
---
```

## Tag Taxonomy

### Level 1: Note Type (pick one)
- `#inbox` — Unprocessed
- `#architecture` — Code-related documentation
- `#reference` — Non-code knowledge

### Level 2: Domain (pick one)
- `#storefront` `#pos` `#backend` `#design` `#ops`

### Level 3: Topic (optional)
- `#auth` `#catalog` `#cart` `#checkout` `#payments`
- `#inventory` `#orders` `#seo` `#search` `#cms`

## Safety

- Do not move or delete files. Only rewrite frontmatter in place.
- Preserve all wiki links (`[[...]]`) and note body content unchanged.
