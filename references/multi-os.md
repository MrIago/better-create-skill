# Multi-OS portability (Windows / macOS / Linux)

A skill others install must run beyond your machine. The common breakers:

## 1. `python3` vs `python` (the #1 real bug)

On **Windows**, the command is usually `python` — `python3` is the Microsoft
Store stub and won't run your script. On macOS/Linux it's `python3`.

In SKILL.md, instruct the model explicitly:
> Use `python3` on macOS/Linux and `python` on Windows for the commands below.

Or detect once at the top of the skill with a `!` block and reuse the result,
or have a tiny launcher. Simplest is the instruction above — the model adapts.

## 2. Never hardcode `/tmp` or `~/...` paths

Use the stdlib, which resolves per-OS:

```python
import tempfile
from pathlib import Path

work = Path(tempfile.mkdtemp(prefix="myskill-"))   # /tmp, C:\...\Temp, /var/folders
cfg  = Path.home() / ".config" / "myskill"          # works on all three
```

`tempfile.gettempdir()` for a shared temp file path; `Path.home()` for user dirs.

## 3. GPU / no-GPU

If you use a local model (e.g. faster-whisper with CUDA), **always fall back**:

```python
try:
    model = WhisperModel(MODEL, device="cuda", compute_type="int8_float16")
except Exception:
    model = WhisperModel(MODEL, device="cpu", compute_type="int8")  # works anywhere, slower
```

And warn the user when on the slow path so they understand:
> No GPU detected — local transcription on CPU is slow. Set an API key (see
> secrets-and-config.md) to offload it.

Best practice: offer an **API alternative** for the no-GPU majority, so the skill
is usable without special hardware. Local-first for those who have it, API for
everyone else.

## 4. Platform-only dependencies

Some deps are OS-specific — document them as such instead of requiring globally.
Example: reading Chrome's encrypted cookies needs `secretstorage` **only on
Linux**; macOS/Windows yt-dlp read them without it. Mark optional deps with the
OS they apply to in your dependency check.

## 5. A preflight check helps everyone

Ship a `scripts/check.py` that verifies required tools and prints the exact
install command per missing one, so a misconfigured machine gets a clear message
instead of a mid-run crash:

```python
import shutil, sys
REQ = [("yt-dlp","pipx install yt-dlp"), ("ffmpeg","brew/apt install ffmpeg")]
missing = [(n,h) for n,h in REQ if not shutil.which(n)]
for n,h in missing: print(f"[check] missing: {n} — {h}", file=sys.stderr)
sys.exit(1 if missing else 0)
```
