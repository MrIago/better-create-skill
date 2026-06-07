# User secrets & config (API keys that work on *their* machine)

If your skill needs API keys (or per-user settings like which browser to read
cookies from), it must be configurable by whoever installs it — without editing
shell profiles. The pattern below lets a non-technical user just tell Claude
"my key is X" and have it persist.

## The pattern: env var OR a persistent `.env`

Read each setting from the **environment first**, then a persistent file at
`~/.config/<skill>/.env`. Env var wins (for people who manage their own), the
file is the easy persistent fallback.

```python
# lib/config.py
import os
from pathlib import Path

CONFIG_FILE = Path.home() / ".config" / "myskill" / ".env"

def _read_file():
    if not CONFIG_FILE.exists():
        return {}
    out = {}
    for line in CONFIG_FILE.read_text(encoding="utf-8").splitlines():
        line = line.strip()
        if not line or line.startswith("#") or "=" not in line:
            continue
        k, _, v = line.partition("=")
        v = v.strip().strip('"').strip("'")
        out[k.strip()] = v
    return out

def get(name, default=None):
    v = os.environ.get(name)
    if v and v.strip():
        return v.strip()
    v = _read_file().get(name)
    return v.strip() if v and v.strip() else default

def set_values(pairs: dict):
    CONFIG_FILE.parent.mkdir(parents=True, exist_ok=True)
    cur = _read_file(); cur.update({k: v for k, v in pairs.items() if v})
    CONFIG_FILE.write_text(
        "# myskill config — secrets, keep private\n"
        + "".join(f"{k}={v}\n" for k, v in cur.items()), encoding="utf-8")
    try: CONFIG_FILE.chmod(0o600)   # best-effort; Windows may ignore
    except OSError: pass
```

Then in your scripts, read settings via `config.get("GROQ_API_KEY")` instead of
`os.environ.get(...)` directly — so both sources work. Read keys **through
`config.get` at point of use**, not only at import time, so the file is honored.

## Configurable via chat

Expose a tiny CLI so the model can save a key the user pastes in conversation:

```bash
python3 "${CLAUDE_SKILL_DIR}/scripts/lib/config.py" GROQ_API_KEY=gsk_...
python3 "${CLAUDE_SKILL_DIR}/scripts/lib/config.py"            # list, masked
```

In SKILL.md, tell the model: *"If the user provides an API key, save it with
config.py; never print it back."*

## Cascade / graceful default

Make the skill work with **whatever** the user has, choosing automatically:

```python
def choose_backend():
    pref = config.get("MYSKILL_BACKEND", "auto")
    if pref != "auto": return pref
    if config.get("GROQ_API_KEY"):   return "groq"     # cloud, no GPU needed
    if config.get("OPENAI_API_KEY"): return "openai"
    return "local"                                      # offline fallback
```

So: power users with a GPU get free local; everyone else sets one key and it
just works; nobody is blocked.

## Hygiene

- `.gitignore` the runtime artifacts (exported cookie files, any `.env` in the
  repo) so secrets never get committed:
  ```
  *-cookies.txt
  .env
  __pycache__/
  ```
- `chmod 0600` the config file. Never echo a key to stdout/stderr/logs.
- Mask when listing: `key[:4] + "…" + key[-4:]`.
