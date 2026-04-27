# work-plugin-marketplace

Shared local marketplace for Claude Code and Codex work plugins.

## Contents

- `zero-powers` from `https://github.com/3219378872/zero-powers`
- Source snapshot: local `origin/main` at `fd608247bdf66b46867bef36a3e212d0ebe27e24`
- Plugin path: `plugins/zero-powers`

## Claude Code

Register this marketplace:

```bash
claude plugin marketplace add /home/bt/projects/skill/work-plugin-marketplace
```

Install the plugin:

```bash
claude plugin install zero-powers@work-plugin-marketplace
```

Claude reads `.claude-plugin/marketplace.json` and installs `zero-powers` from `./plugins/zero-powers`.

## Codex

Register this marketplace:

```bash
codex plugin marketplace add /home/bt/projects/skill/work-plugin-marketplace
```

Codex reads `.agents/plugins/marketplace.json` and discovers `zero-powers` from `./plugins/zero-powers`.

## Maintenance

Refresh the vendored plugin snapshot from the local source clone:

```bash
git -C /home/bt/projects/skill/zero-powers fetch origin
git -C /home/bt/projects/skill/zero-powers archive --format=tar --output=/tmp/zero-powers-origin-main.tar origin/main
tar -xf /tmp/zero-powers-origin-main.tar -C plugins/zero-powers
```

After refreshing, keep `plugins/zero-powers/.codex-plugin/plugin.json` and root marketplace files in place.
