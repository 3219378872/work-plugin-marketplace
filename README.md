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
