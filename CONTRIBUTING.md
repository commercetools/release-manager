# Contributing

This is a commercetools-labs project, provided as-is and unsupported (see [SUPPORT.md](SUPPORT.md)).
Contributions are welcome and reviewed on a best-effort basis.

Only contribute code you have the right to license under MIT. Never commit credentials or
personal data; use obviously synthetic fixtures such as `jane@example.com`.

## Before you open a PR

One command, from the repo root, covering every package:

```bash
npm run install:all   # first time only
npm run predeploy
```

That is the service's unit tests, then typecheck and lint across the console and both
Merchant Center apps, then a production build of the console. It runs in well under a
minute. The repository has no CI, so `predeploy` is the whole gate: please run it before
you open a PR.

`npm run format` inside `packages/mc-releases` or `packages/mc-branch-editor` fixes most of
their lint failures, which are prettier formatting.

**The end-to-end suite sits outside that gate.** `npm run test:e2e` talks to two real
commercetools projects, creates and deletes data in both, and takes minutes rather than
milliseconds. Run it when you have changed the deploy path, against projects you are
willing to have written to; see
[`test/README-e2e.md`](packages/release-deploy/test/README-e2e.md). It refuses to run
unless the resolved project key matches `E2E_STAGE_PROJECT`, which is what stands between
a stray `RUN_E2E=1` and somebody's production catalog. Please keep that guard as it is, and
keep the suite out of `predeploy`.

The same goes for `packages/mc-releases/test/ui`: a Playwright smoke test over a standalone
harness page, run on demand with `npx playwright test test/ui`. Playwright is deliberately
not a dependency of that app.

## What good looks like here

This repository optimizes for **being read**. People land in it to understand how a pattern
works before writing their own version, so:

- **Comment the *why*, not the *what*.** `// bump the version` is noise. "commercetools
  rejects a stale version with a 409, so the registry uses compare-and-swap and two
  concurrent branch writes never silently drop one" is the reason the code is shaped the way
  it is, and it cannot be recovered from reading it.
- **Match the surrounding code.** Its naming, its comment density, its idioms. A PR that
  introduces a second style makes the codebase harder to read even if the new style is
  better in isolation.
- **Keep the service dependency-free.** `packages/release-deploy` has no `node_modules` and
  runs on plain Node ESM. That is a deliberate property: it makes the service auditable and
  its container trivial. Adding a dependency to it needs a real argument.
- **Keep the packages independent.** No workspace hoisting, one lockfile each, no package
  importing another's source. The console and the Merchant Center apps reach the service
  over HTTP, which is the same boundary a real deployment has, so a change that works only
  because two packages share a checkout is a change that breaks in production.
- **The release mechanics live in the service.** Branch, fork, merge, diff, deploy and
  rollback belong in `packages/release-deploy`, and every front end calls them.
  Reimplementing one of them in the console or an MC app puts two state machines in front
  of the same data.
- **Keep it project-agnostic.** Nothing about a particular deployment belongs in the code.
  Projects, credentials and URLs come from the environment; anything an operator should be
  able to change belongs in configuration, not a constant.
- **Treat the container names as frozen.** `release-registry`, `release-branch`,
  `release-acl` and friends are the wire contract between the service, the console and both
  MC apps. Renaming one is a data migration, and it has to land in all four at once, plus
  `bin/migrate-containers.mjs`. The console mirrors them in two files (`lib/containers.ts`
  for the app, `tools/containers.mjs` for the operator scripts, which run outside the
  Next.js build and cannot import TypeScript); both have to agree.
- **Anything that writes to live needs a test.** The approval gate, the merge, the deploy
  and the rollback all have unit coverage that runs without credentials. If you touch them,
  extend it alongside your change rather than relying on a manual run against a project.

## Reporting a security issue

Security issues in this code go in a GitHub issue; the repository has no private disclosure
channel, so if one is genuinely sensitive, please bear that in mind before you file it.
Vulnerabilities in the commercetools platform itself go to
[commercetools' security contact](https://commercetools.com/privacy-and-security).

By contributing you agree your contribution is licensed under the [MIT License](LICENSE).
