# Anatomy of a skill

## Directory layout

```
my-skill/
├── SKILL.md                 # required — frontmatter + instructions
├── references/              # docs loaded ON DEMAND (one per variant/topic)
│   ├── aws.md
│   └── gcp.md
├── scripts/                 # executable code — RUN, not loaded into context
│   └── do_thing.py
└── assets/                  # files used in output (templates, fonts, icons)
```

Only `SKILL.md` is required. Everything else is optional and exists to keep
SKILL.md small.

## Progressive disclosure — the core idea

Three levels, each loaded later and cheaper-until-needed:

| Level | What | When in context | Budget |
|-------|------|-----------------|--------|
| 1. Metadata | `name` + `description` | **always** | ~100 words |
| 2. SKILL.md body | the instructions | when the skill triggers | keep < 500 lines |
| 3. Bundled files | `references/*`, `scripts/*`, `assets/*` | only when read/run | unlimited |

Design so **level 1 is enough to trigger correctly** and **level 2 handles the
common case**, pushing depth to level 3. A reference file the model never needs
costs nothing.

## What goes where

- **Reference content** (conventions, API details, per-variant docs) → `references/`,
  pointed to from SKILL.md with a one-line "read this when…".
- **Deterministic/repeatable work** → `scripts/` (see scripts-pattern.md).
- **Output materials** (a docx template, a logo) → `assets/`.
- **The decision logic / routing / when-to-do-what** → SKILL.md body.

## Domain organization (multi-variant skills)

When a skill spans platforms/frameworks/clouds, route in SKILL.md and put one
reference per variant. The model reads only the relevant one:

```
cloud-deploy/
├── SKILL.md            # "detect the provider, then read references/<provider>.md"
└── references/{aws,gcp,azure}.md
```

## Keep SKILL.md lean

The body stays in context for the whole session once loaded — every line is a
recurring cost. If you pass ~500 lines, that's the signal to add a layer:
move detail into `references/` and leave a pointer. Apply the same conciseness
test you'd use for a good CLAUDE.md.

## Reference files >300 lines

Add a short table of contents at the top so the model can jump to the section it
needs without reading the whole thing.
