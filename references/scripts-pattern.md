# Bundling scripts

## When to bundle a script (vs inline instructions)

Bundle a script when the work is **deterministic and repeatable** — the model
would otherwise rewrite the same helper on every invocation. Signals:

- Parsing/transforming a fixed format, calling an API the same way, file ops,
  anything with exact flags that must be right every time.
- You notice (in testing) that every run independently writes a similar
  `do_thing.py`. That's the cue to write it once and bundle it.

Keep inline (no script) when the value is judgement/prose the model does live.

## How the skill calls a bundled script

Always reference via `${CLAUDE_SKILL_DIR}` so it resolves wherever the skill is
installed (personal, project, or plugin cache):

```markdown
Run the extractor:

```bash
python3 "${CLAUDE_SKILL_DIR}/scripts/extract.py" "<arg>"
```
```

Have the script **print a clear result to stdout** that the model reads (paths,
status, data). Diagnostics go to **stderr** so they don't pollute the result.

## Make scripts self-describing & robust

- Top docstring: what it does, usage, and that "its stdout is the result".
- Exit non-zero with a one-line actionable error on failure (e.g. "ffmpeg not
  installed — `brew install ffmpeg`"), not a raw traceback.
- Handle `SIGPIPE` so `| head` doesn't dump a BrokenPipeError:
  ```python
  if __name__ == "__main__":
      try:
          import signal; signal.signal(signal.SIGPIPE, signal.SIG_DFL)
      except (ImportError, AttributeError, ValueError):
          pass
      raise SystemExit(main())
  ```
- Prefer stdlib / pure HTTP over heavy SDKs when feasible — fewer install
  requirements for whoever installs the skill.

## Per-domain scripts: duplicate first, DRY empirically

When a skill has several variants (platforms, providers), it's fine — often
better — to start with **self-contained scripts per variant**, even with
duplicated code. You don't yet know what's truly shared; abstracting early
creates a "core" that doesn't fit every variant's exceptions.

Once 2–3 variants exist and you can SEE the real repetition, extract just that
into a shared `scripts/lib/` module the variants import. Let evidence, not
prediction, drive the DRY pass.

## Parallelism is a free speedup

Independent script work (N frames, N files, N API calls within rate limits)
should run concurrently, not in a loop:

```python
import concurrent.futures
with concurrent.futures.ThreadPoolExecutor(max_workers=8) as ex:
    results = list(ex.map(do_one, items))
```

Watch the constraint: parallelize I/O and independent HTTP freely (respect the
provider's req/min); do NOT parallelize things that share a scarce resource
(e.g. one GPU model in VRAM — load it once, reuse it sequentially).

## A "batch" entry point pays off

If users will run the same op over many targets (e.g. transcribe every video in
a profile), accept multiple inputs in one invocation so expensive setup (model
load, auth) happens once and the rest parallelizes — far faster than the model
calling the script once per item.
