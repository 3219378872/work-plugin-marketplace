# GitHub Source Work Plugin Marketplace Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert this repository from a vendored plugin snapshot into a GitHub-hosted marketplace and reference index that resolves `zero-powers` from `3219378872/zero-powers`.

**Architecture:** The root marketplace manifests remain the only plugin registration surface in this repository. Claude Code and Codex each get a platform-specific manifest that references the plugin source repository through a GitHub source object, while the vendored `plugins/zero-powers` snapshot is removed. `README.md` becomes the operator-facing guide for registering, updating, and extending this marketplace.

**Tech Stack:** JSON marketplace manifests, Markdown documentation, Git, Claude Code plugin marketplace CLI, Codex plugin marketplace CLI.

---

## File Structure

- Modify: `.claude-plugin/marketplace.json`
  - Responsibility: Claude Code marketplace metadata, owner identity, and GitHub source entry for `zero-powers`.
- Modify: `.agents/plugins/marketplace.json`
  - Responsibility: Codex marketplace metadata and GitHub source entry for `zero-powers`.
- Modify: `README.md`
  - Responsibility: Explain this repository as the GitHub entry point for work plugin and script references, with concrete Claude/Codex commands.
- Delete: `plugins/zero-powers/`
  - Responsibility removed: vendored plugin source cache. The plugin source lives in `3219378872/zero-powers`.
- Leave unchanged: `docs/superpowers/specs/2026-04-27-github-source-work-plugin-marketplace-design.md`
  - Responsibility: Approved design source for this implementation.

## Task 1: Baseline Desired-State Check

**Files:**
- No file changes

- [ ] **Step 1: Run the desired-state validator before edits**

Run:

```bash
python3 - <<'PY'
import json
import pathlib
import sys

root = pathlib.Path(".")
errors = []

claude = json.loads((root / ".claude-plugin/marketplace.json").read_text(encoding="utf-8"))
codex = json.loads((root / ".agents/plugins/marketplace.json").read_text(encoding="utf-8"))

github_source = {"source": "github", "repo": "3219378872/zero-powers"}
owner = {"name": "风起", "email": "3219378872@qq.com"}

if claude.get("owner") != owner:
    errors.append("claude marketplace owner is not 风起 <3219378872@qq.com>")

if claude.get("plugins", [{}])[0].get("source") != github_source:
    errors.append("claude zero-powers source is not the GitHub source object")

if codex.get("plugins", [{}])[0].get("source") != github_source:
    errors.append("codex zero-powers source is not the GitHub source object")

if (root / "plugins" / "zero-powers").exists():
    errors.append("vendored plugins/zero-powers cache still exists")

readme = (root / "README.md").read_text(encoding="utf-8")
for stale in [
    "Source snapshot",
    "Plugin path: `plugins/zero-powers`",
    "./plugins/zero-powers",
    "Refresh the vendored plugin snapshot",
]:
    if stale in readme:
        errors.append(f"README still contains stale text: {stale}")

if errors:
    print("FAIL")
    for error in errors:
        print(f"- {error}")
    sys.exit(1)

print("PASS")
PY
```

Expected: FAIL with messages for the current local source references, stale README text, and existing `plugins/zero-powers` directory.

## Task 2: Update Marketplace Manifests To GitHub Sources

**Files:**
- Modify: `.claude-plugin/marketplace.json`
- Modify: `.agents/plugins/marketplace.json`

- [ ] **Step 1: Replace `.claude-plugin/marketplace.json`**

Replace the entire file with:

```json
{
  "name": "work-plugin-marketplace",
  "owner": {
    "name": "风起",
    "email": "3219378872@qq.com"
  },
  "metadata": {
    "description": "GitHub-hosted marketplace and reference index for work plugins and scripts."
  },
  "plugins": [
    {
      "name": "zero-powers",
      "source": {
        "source": "github",
        "repo": "3219378872/zero-powers"
      }
    }
  ]
}
```

- [ ] **Step 2: Replace `.agents/plugins/marketplace.json`**

Replace the entire file with:

```json
{
  "name": "work-plugin-marketplace",
  "interface": {
    "displayName": "Work Plugin Marketplace"
  },
  "plugins": [
    {
      "name": "zero-powers",
      "source": {
        "source": "github",
        "repo": "3219378872/zero-powers"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Coding"
    }
  ]
}
```

- [ ] **Step 3: Verify both manifests parse as JSON**

Run:

```bash
python3 -m json.tool .claude-plugin/marketplace.json
```

Expected: formatted JSON is printed and the command exits with status 0.

Run:

```bash
python3 -m json.tool .agents/plugins/marketplace.json
```

Expected: formatted JSON is printed and the command exits with status 0.

- [ ] **Step 4: Run platform manifest validation where available**

Run:

```bash
claude plugin validate .claude-plugin/marketplace.json
```

Expected: Claude reports the marketplace manifest is valid or exits with status 0.

Run:

```bash
codex plugin marketplace add --help
```

Expected: help text includes `Supports owner/repo[@ref], HTTP(S) Git URLs, SSH URLs, or local marketplace root directories`.

- [ ] **Step 5: Commit manifest changes**

Run:

```bash
git add .claude-plugin/marketplace.json .agents/plugins/marketplace.json
git commit -m "chore: reference zero-powers from github"
```

Expected: commit succeeds with the two marketplace manifest changes.

## Task 3: Rewrite README As The Work Reference Entry Point

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Replace `README.md`**

Replace the entire file with:

```markdown
# work-plugin-marketplace

GitHub-hosted marketplace and reference index for work plugins and scripts.

Maintainer: 风起 <3219378872@qq.com>

## Purpose

This repository is the top-level entry point for work plugin and script references. It stores
marketplace manifests and documentation, not plugin source snapshots.

Plugin source code stays in its own GitHub repository and is resolved by the marketplace at install
or update time.

## Plugins

| Plugin | Source | Purpose |
| --- | --- | --- |
| `zero-powers` | `https://github.com/3219378872/zero-powers` | go-zero microservice development skills, patterns, and review workflows |

## Claude Code

Register this marketplace from GitHub:

```bash
claude plugin marketplace add 3219378872/work-plugin-marketplace
```

List configured marketplaces:

```bash
claude plugin marketplace list
```

Install `zero-powers` from this marketplace:

```bash
claude plugin install zero-powers@work-plugin-marketplace
```

Update the marketplace and installed plugin when the source repository changes:

```bash
claude plugin marketplace update work-plugin-marketplace
claude plugin update zero-powers
```

## Codex

Register this marketplace from GitHub:

```bash
codex plugin marketplace add 3219378872/work-plugin-marketplace
```

Refresh this marketplace when plugin source repositories change:

```bash
codex plugin marketplace upgrade work-plugin-marketplace
```

Codex reads `.agents/plugins/marketplace.json` and resolves `zero-powers` from
`3219378872/zero-powers`.

## Maintenance

Add new work plugins by editing both marketplace manifests:

- `.claude-plugin/marketplace.json`
- `.agents/plugins/marketplace.json`

Use GitHub source references for plugin entries:

```json
{
  "source": {
    "source": "github",
    "repo": "OWNER/REPO"
  }
}
```

Do not vendor plugin source directories into this repository. Keep plugin implementation,
release tags, and plugin-specific metadata in the plugin source repository.

If a plugin must be pinned for reproducibility, add the explicit version or ref for that plugin and
document the reason in this README.
```

- [ ] **Step 2: Check the README no longer describes vendored cache maintenance**

Run:

```bash
rg -n "Source snapshot|Plugin path: `plugins/zero-powers`|./plugins/zero-powers|Refresh the vendored plugin snapshot" README.md
```

Expected: no matches and exit status 1 from `rg`.

- [ ] **Step 3: Commit README changes**

Run:

```bash
git add README.md
git commit -m "docs: document github source marketplace"
```

Expected: commit succeeds with the README rewrite.

## Task 4: Remove Vendored Plugin Snapshot

**Files:**
- Delete: `plugins/zero-powers/`

- [ ] **Step 1: Remove the vendored plugin directory from git**

Run:

```bash
git rm -r plugins/zero-powers
```

Expected: git lists deleted files under `plugins/zero-powers`.

- [ ] **Step 2: Verify the snapshot directory no longer exists**

Run:

```bash
test ! -e plugins/zero-powers
```

Expected: no output and exit status 0.

- [ ] **Step 3: Verify no tracked files remain under `plugins/zero-powers`**

Run:

```bash
git ls-files plugins/zero-powers
```

Expected: no output.

- [ ] **Step 4: Commit the deletion**

Run:

```bash
git commit -m "chore: remove vendored zero-powers cache"
```

Expected: commit succeeds with deletions under `plugins/zero-powers`.

## Task 5: Final Verification And Integration Commit Check

**Files:**
- Verify: `.claude-plugin/marketplace.json`
- Verify: `.agents/plugins/marketplace.json`
- Verify: `README.md`
- Verify: git index

- [ ] **Step 1: Run the desired-state validator again**

Run:

```bash
python3 - <<'PY'
import json
import pathlib
import sys

root = pathlib.Path(".")
errors = []

claude = json.loads((root / ".claude-plugin/marketplace.json").read_text(encoding="utf-8"))
codex = json.loads((root / ".agents/plugins/marketplace.json").read_text(encoding="utf-8"))

github_source = {"source": "github", "repo": "3219378872/zero-powers"}
owner = {"name": "风起", "email": "3219378872@qq.com"}

if claude.get("owner") != owner:
    errors.append("claude marketplace owner is not 风起 <3219378872@qq.com>")

if claude.get("plugins", [{}])[0].get("source") != github_source:
    errors.append("claude zero-powers source is not the GitHub source object")

if codex.get("plugins", [{}])[0].get("source") != github_source:
    errors.append("codex zero-powers source is not the GitHub source object")

if (root / "plugins" / "zero-powers").exists():
    errors.append("vendored plugins/zero-powers cache still exists")

readme = (root / "README.md").read_text(encoding="utf-8")
for stale in [
    "Source snapshot",
    "Plugin path: `plugins/zero-powers`",
    "./plugins/zero-powers",
    "Refresh the vendored plugin snapshot",
]:
    if stale in readme:
        errors.append(f"README still contains stale text: {stale}")

if errors:
    print("FAIL")
    for error in errors:
        print(f"- {error}")
    sys.exit(1)

print("PASS")
PY
```

Expected: `PASS`.

- [ ] **Step 2: Search for stale local marketplace source references**

Run:

```bash
rg -n '"source": "local"|plugins/zero-powers|fd608247bdf66b46867bef36a3e212d0ebe27e24|go-zero Community|go-zero@go-zero.dev' README.md .claude-plugin .agents
```

Expected: no matches and exit status 1 from `rg`.

- [ ] **Step 3: Verify marketplace CLI command availability**

Run:

```bash
claude plugin marketplace add --help
```

Expected: help text says the source can be a URL, path, or GitHub repo.

Run:

```bash
claude plugin marketplace update --help
```

Expected: help text says it updates marketplace sources.

Run:

```bash
codex plugin marketplace add --help
```

Expected: help text says the source supports `owner/repo[@ref]`.

Run:

```bash
codex plugin marketplace upgrade --help
```

Expected: help text for upgrading a marketplace is printed.

- [ ] **Step 4: Confirm worktree cleanliness**

Run:

```bash
git status --short
```

Expected: no output.

- [ ] **Step 5: Record final commit range for handoff**

Run:

```bash
git log --oneline --max-count=5
```

Expected: recent commits include:

```text
chore: remove vendored zero-powers cache
docs: document github source marketplace
chore: reference zero-powers from github
docs: design github source plugin marketplace
```
