# GitHub Source Work Plugin Marketplace Design

## Context

This repository is currently a shared Claude Code and Codex plugin marketplace. It vendors
`zero-powers` under `plugins/zero-powers` and points both marketplace manifests at that local copy.

The new direction is to make this repository the central GitHub-hosted entry point for work
plugin and script references, not a cache of plugin source code. Plugin contents stay in their
own GitHub repositories and are resolved from those repositories when installed or upgraded.

Primary source format reference:

- Claude plugin marketplace docs: https://code.claude.com/docs/zh-CN/discover-plugins
- Claude marketplace authoring docs: https://code.claude.com/docs/zh-CN/plugin-marketplaces

## Goals

- Remove vendored plugin caches from this repository.
- Point marketplace manifests at the latest plugin source through GitHub source references.
- Standardize repository and plugin author metadata to `风起 <3219378872@qq.com>`.
- Reposition this repository as the top-level GitHub repository for work plugin and script
  references.
- Keep future plugin/script additions lightweight, auditable, and easy to upgrade.

## Non-Goals

- Do not maintain plugin source code in this repository.
- Do not pin plugin entries to a fixed commit unless a future plugin needs reproducible freeze
  behavior.
- Do not redesign the internal content of `zero-powers`.
- Do not install or upgrade local user plugin state as part of the repository change.

## Architecture

The repository becomes a source-of-truth marketplace and index:

- `.claude-plugin/marketplace.json` defines the Claude Code marketplace.
- `.agents/plugins/marketplace.json` defines the Codex marketplace.
- `README.md` explains how to add the marketplace from GitHub, install plugins, and upgrade them.
- Future `scripts/` or `references/` content may be added as top-level indexed work assets, but
  plugin implementation files do not live here.

`plugins/zero-powers` is removed because it is a cached snapshot. The marketplace entry for
`zero-powers` resolves from `3219378872/zero-powers` instead.

## Marketplace Entries

Claude marketplace entry target:

```json
{
  "name": "zero-powers",
  "source": {
    "source": "github",
    "repo": "3219378872/zero-powers"
  }
}
```

Codex marketplace entry target:

```json
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
```

Both entries intentionally omit a fixed version/ref so the latest plugin source can be located by
the platform's marketplace upgrade flow.

## Metadata

All maintained marketplace-level author or owner metadata should use:

```json
{
  "name": "风起",
  "email": "3219378872@qq.com"
}
```

If plugin-specific metadata remains in this repository, it should use the same author identity.
After removing vendored plugin contents, plugin-specific metadata is expected to live in the
plugin source repository.

## Data Flow

1. User registers this repository as a marketplace from GitHub.
2. Claude Code or Codex reads the marketplace manifest.
3. The selected plugin entry resolves its source from `3219378872/zero-powers`.
4. The tool installs or upgrades the plugin from the plugin source repository.
5. Future plugin updates happen in the plugin repository, then become available through marketplace
   upgrade/update commands.

## Error Handling

- If a GitHub source is unavailable, the tool should fail at install or upgrade time; the README
  should tell users to verify network access and repository visibility.
- If a platform does not support GitHub plugin sources with the same manifest shape, keep the
  platform-specific manifest separate rather than reintroducing vendored plugin caches.
- If reproducibility is required for a specific plugin, add an explicit version/ref for that plugin
  only and document why it is pinned.

## Testing And Verification

Repository verification should include:

- Parse `.claude-plugin/marketplace.json` as JSON.
- Parse `.agents/plugins/marketplace.json` as JSON.
- Search for stale vendored snapshot wording such as `Source snapshot`, `plugins/zero-powers`, and
  fixed source commit hashes in root documentation.
- Confirm `plugins/zero-powers` is removed from the worktree and git index.
- Confirm author metadata in maintained manifests is `风起 <3219378872@qq.com>`.

Manual verification should include:

- `claude plugin marketplace add 3219378872/work-plugin-marketplace`
- `codex plugin marketplace add 3219378872/work-plugin-marketplace`
- Install or upgrade `zero-powers` from the registered marketplace.

## Implementation Scope

The implementation plan should make these changes:

1. Delete `plugins/zero-powers` from this repository.
2. Update `.claude-plugin/marketplace.json` to GitHub source references and owner metadata.
3. Update `.agents/plugins/marketplace.json` to GitHub source references.
4. Rewrite `README.md` around GitHub marketplace registration, latest plugin resolution, and future
   work script references.
5. Run JSON and stale-reference checks.
6. Commit the repository changes after verification.
