# Changesets

Feature PRs that change core package behavior should include a changeset:

```bash
pnpm changeset
```

This rollout only versions the core packages: `@0xkoller/demokit`,
`@0xkoller/create-demokit-app`, and `@0xkoller/init-demokit`. Plugin packages
(e.g. `@demokit-dev/plugin`) remain on their existing manual publish workflows and are ignored
by Changesets.
