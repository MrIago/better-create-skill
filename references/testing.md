# Testing & iterating on a skill

## Validate first

```bash
python3 .../skill-creator/scripts/quick_validate.py <skill-dir>
```
Or check by hand: allowed frontmatter keys only, kebab-case name ≤64, description
≤1024 with no `<`/`>`. Then syntax-check any bundled scripts
(`python3 -c "import ast; ast.parse(open('x.py').read())"`).

## Test the triggering + the behavior

Two different things to test:

1. **Does it trigger?** Write 8–10 realistic prompts a user would actually type
   (specific, with context — file paths, real phrasing, some casual/typo'd) that
   *should* trigger, and 8–10 near-misses that *should not* (share keywords but
   need something else). If it under/over-triggers, tune the `description`.

2. **Does it do the job?** Run it on real inputs. If you have subagents, run the
   same prompt with and without the skill (baseline) to see what it adds.

## Test the scripts manually, one backend/path at a time

Before wiring a script into the skill, run it standalone on real data and verify
the actual output — don't assume. Especially: each external backend/provider,
each OS-specific path, and edge cases (empty result, long input, auth missing).
Real example lessons that only surfaced by testing:
- An API that returns text but silently **drops timestamps** → unusable for a
  use case that needs them. Verify the *shape* of the response, not just 200 OK.
- A "video" with audio that's actually **music/silent** → transcription empty is
  correct, not a bug. Confirm with the real file before concluding.

## Iterate on the skill, not on the examples

You iterate on a few examples for speed, but the skill must generalize. If a fix
only helps your test cases, it's overfit. Prefer reframing/explaining-why over
piling on rigid rules. Remove instructions that don't pull their weight — read
the transcript and cut anything that made the model waste steps.

## Install-test the real thing

The ultimate test is installing it as a plugin in a fresh session and using it
with no prior context (see publishing.md). If the command appears and works
cold, it's done.
