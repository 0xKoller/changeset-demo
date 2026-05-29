# changeset-demo

A standalone, self-contained monorepo that **replicates the [Changesets](https://github.com/changesets/changesets) release setup we added to xmcp** — so you can inspect and run it without the noise of the full xmcp codebase.

It mirrors the real setup 1:1 (only names/paths are swapped): a **fixed core group** of three packages versioned in lock-step, one **ignored plugin** on a separate manual flow, plus both the **stable** and **canary** GitHub publish workflows.

> [!NOTE]
> The core packages are published under the **`@0xkoller`** npm scope (the publisher's personal
> username scope — no org required). The local versioning demo needs nothing configured; a real
> `npm publish` only needs you to be logged in as `0xkoller` (or an `NPM_TOKEN` for that account in
> CI) and `--access public` (already set via each package's `publishConfig`).

## How it maps to xmcp

| xmcp (real)           | this demo                            | role                          |
| --------------------- | ------------------------------------ | ----------------------------- |
| `xmcp`                | `@0xkoller/demokit`           | core (fixed group)            |
| `create-xmcp-app`     | `@0xkoller/create-demokit-app`| core (fixed group)            |
| `init-xmcp`           | `@0xkoller/init-demokit`      | core (fixed group)            |
| `@xmcp-dev/*` plugins | `@demokit-dev/plugin`                | ignored (manual publish flow) |

- **Fixed group** (`.changeset/config.json` → `fixed`): the three `@0xkoller/*` core packages always bump to the **same** version together.
- **Ignored** (`.changeset/config.json` → `ignore`): `@demokit-dev/plugin` is excluded from Changesets *versioning*. It is also marked `"private": true` so `changeset publish` skips it — modeling a plugin on its own separate/manual publish flow. (Note: the `ignore` list only affects versioning; `private: true` is what keeps the shared publish from touching it.)

## Key files (faithful copies of xmcp's)

- `.changeset/config.json` — fixed group + ignore + `access`, `baseBranch`, `updateInternalDependencies`, `privatePackages`.
- `.changeset/README.md` — contributor guide.
- `package.json` — the three scripts: `changeset`, `version-packages` (`changeset version`), `publish-packages` (`changeset publish`).
- `.github/workflows/publish.yml` — **stable** release: on `main` push uses `changesets/action@v1` to open a "Version Packages" PR / publish; manual dispatch is dry-run only.
- `.github/workflows/publish-canary.yml` — **canary** release: on `canary` push derives an incremental `X.Y.Z-canary.N` version, builds, verifies, and publishes with the `canary` dist-tag; manual dispatch is dry-run only.

## Run the demo locally

No npm or GitHub access required — Changesets only reads the workspace `package.json` files.

```bash
pnpm install
pnpm build            # turbo build — produces dist/ + index.js artifacts

# 1. Author a changeset (interactive: pick a bump, e.g. minor on demokit)
pnpm changeset

# 2. Apply it
pnpm version-packages
```

After step 2, observe:

- **All three** core packages (`@0xkoller/demokit`, `@0xkoller/create-demokit-app`, `@0xkoller/init-demokit`) bump to the **same** new version — that's the `fixed` group.
- Each core package gets a `CHANGELOG.md`.
- `@demokit-dev/plugin` is **untouched** — that's the `ignore` list.
- The applied `.changeset/*.md` file is consumed (removed).

Inspect the result:

```bash
git diff
git status
```

## Publishing (workflows)

The two workflows are faithful copies of xmcp's. To publish for real they require an `NPM_TOKEN`
repository secret; manual `workflow_dispatch` runs are gated behind `dry_run=true` so nothing is
published by accident. Stable releases happen on push to `main`; canary releases on push to `canary`.
