# Publishing a skill as an installable plugin

This is the part the official skill-creator skips. Goal: anyone can run
`/plugin marketplace add you/repo` then `/plugin install`, and get your `/command`.

## Two layouts

**A. The repo IS one skill** (recommended for a standalone skill — popular,
clean, good for a portfolio). `SKILL.md` at the repo root + `.claude-plugin/`
with BOTH `plugin.json` and `marketplace.json`:

```
my-skill/                      (the GitHub repo)
├── SKILL.md                   # name: in frontmatter becomes the command
├── README.md
├── LICENSE
├── .gitignore
├── scripts/ …                 # if any
└── .claude-plugin/
    ├── plugin.json
    └── marketplace.json       # source: "./" — repo root is the plugin
```

**B. A marketplace of many skills** — one repo, `.claude-plugin/marketplace.json`
at the root listing each plugin in its own subfolder (`source: "./name"`), each
subfolder with its own `.claude-plugin/plugin.json` and `skills/<name>/SKILL.md`.
Use this to grow a personal collection.

## plugin.json (per plugin)

```json
{
  "name": "my-skill",
  "version": "0.1.0",
  "description": "What it does + when. (mirrors SKILL.md description)",
  "author": { "name": "Your Name" },
  "repository": "https://github.com/you/my-skill",
  "homepage": "https://github.com/you/my-skill",
  "license": "MIT",
  "keywords": ["..."]
}
```

## marketplace.json (layout A — repo is one plugin)

```json
{
  "name": "my-skill",
  "metadata": { "description": "One-line pitch of the skill." },
  "owner": { "name": "Your Name" },
  "plugins": [
    {
      "name": "my-skill",
      "description": "What it does + when.",
      "author": { "name": "Your Name" },
      "source": "./",
      "category": "productivity",
      "homepage": "https://github.com/you/my-skill"
    }
  ]
}
```

`source: "./"` means "the plugin is this repo root". For a multi-skill
marketplace, list several entries with `source: "./subfolder"`.

## README — set expectations honestly

State up front: it's a **Claude Code (local) skill, not claude.ai web** if it
shells out to local tools/GPU/cookies (the web sandbox has none of those). List
requirements per profile (OS, GPU vs API key, login/cookies). Give the two
install methods (marketplace + manual git clone into `~/.claude/skills/<name>`).

## Publish to GitHub (via API + git)

```bash
# create the repo (public)
curl -s -X POST -H "Authorization: token $GH_TOKEN" \
  -H "Accept: application/vnd.github+json" https://api.github.com/user/repos \
  -d '{"name":"my-skill","description":"…","private":false}'

# init + commit + push (run inside the skill dir = repo root)
git init -q && git branch -M main
git add -A
git -c commit.gpgsign=false commit -q -m "feat: my-skill"
git push -q "https://$GH_TOKEN@github.com/you/my-skill.git" main
git remote set-url origin "https://github.com/you/my-skill.git"   # drop token from remote
```

Verify the marketplace.json is reachable raw:
```bash
curl -s -o /dev/null -w "%{http_code}\n" \
  "https://raw.githubusercontent.com/you/my-skill/main/.claude-plugin/marketplace.json"   # expect 200
```

## Install / update cycle (what the user runs)

```
/plugin marketplace add you/my-skill     # register the marketplace
/plugin install my-skill@my-skill        # install the plugin (name@marketplace)
/reload-plugins                          # apply — the command now exists
```
To ship an update: bump `version` in plugin.json, commit/push, then the user
`/plugin update my-skill@my-skill` + `/reload-plugins`.

## Notes

- The plugin-root `SKILL.md` gets its command from the frontmatter `name`.
- Installed plugins live in `~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`
  — `${CLAUDE_SKILL_DIR}` resolves there, so script paths keep working.
- A `commands/<name>.md` shim used to be needed so the command showed up; on
  current Claude Code a root SKILL.md exposes it directly — test, add the shim
  only if the command doesn't appear.
- Don't ship a `.skill`/web-bundle release workflow for a local-only skill — the
  `.skill` format targets claude.ai web, which can't run local tools anyway.
