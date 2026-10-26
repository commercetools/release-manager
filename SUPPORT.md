# Support

Thanks for taking a look at Release Manager. It is commercetools-labs reference code: shared in
the hope that it is useful, and **unsupported**.

In practice that means:

- It is freely available under the MIT license ([LICENSE](LICENSE)) and is provided as-is, without
  warranty.
- It isn't covered by commercetools Support, your account team or any service-level agreement, so
  we can't promise replies, fixes or new releases.
- Issues and pull requests are welcome, and we'll take a look when we can.

## If you are going to production with this

The deploy service writes to your production project; that is its whole job. Before you ship,
these are the things worth checking:

- **Set `DEPLOY_SERVICE_TOKEN`.** With it unset, the service skips its Bearer check entirely.
  That is convenient for local development, and it leaves every endpoint that writes to
  production open to anyone who can reach the URL. Check it is set on the deployed service,
  not just in your `.env`.
- **Verify which projects you are wired to.** `STAGE_CTP_*` and `LIVE_CTP_*` decide what gets
  written where, and a missing prefixed variable falls back to the un-prefixed `CTP_*`. Two
  consequences follow: with `STAGE_*` unset, stage and live resolve to the *same* project, and
  an unrecognised `?project=` alias resolves to **live** rather than failing. Both happen
  quietly, so read the resolved key back from the service before you trust a deploy.
- **Scope your API clients.** The service needs broad write access on both projects, since it
  serializes from one and upserts into the other. It does not need `manage_api_clients` or
  `manage_project_settings`, so give it a scoped client rather than a full admin one, even
  when an admin client looks like the quickest way past a 403.
- **Know how far back a rollback goes.** A deploy snapshots what it replaced, so a rollback
  restores the state immediately before *that* deploy. It is one step, not an arbitrary point
  in history.
- **Keep the CustomObject container names as they are.** `release-registry`, `release-branch`,
  `release-asset-history`, `release-acl` and `release-config` are a wire contract shared with
  every other reader of the same project. Renaming one leaves the data under it behind rather
  than migrating it. For a deployment that still uses the older `az-*` names,
  `bin/migrate-containers.mjs` copies each object across and leaves the originals in place
  until you tell it to remove them.
- **Set `SESSION_SECRET` and `BOOTSTRAP_ADMIN_EMAIL` on the console, and grant a real admin
  on day one.** The bootstrap admin is how a fresh install gets in, and it stays privileged
  until the access list has an entry, so it is a standing credential until somebody grants
  themselves Admin under **Permissions**.
- **The access list is the roster.** The console's authoring project also holds storefront
  shoppers, and a shopper is a commercetools Customer like any other. A correct password
  admits nobody without an access-list entry. So an empty access list plus a set
  `BOOTSTRAP_ADMIN_EMAIL` is exactly one way in, and removing that variable before anyone
  holds a role locks everyone out.
- **The service and the console share one access list (`release-acl`).** The service treats
  an empty one as fail-open, granting every capability to everyone, so a console that looks
  locked down can sit in front of a service that is not. Check the service's own ACL is
  populated, not just the console's Permissions page.
- **Put the console behind your own authentication if it faces anything but a trusted
  network.** Its sign-in is a commercetools Customer sign-in with a signed-cookie session;
  there is no MFA, no rate limiting on the login route, and no lockout.
- **Add your own observability, rate limiting and audit logging.** The per-release audit
  trail is the only one built in.
- **Pin and audit dependencies.** The service has none by design; the console and the two
  Merchant Center apps have plenty, and keeping them current is up to you.

## If you need help

- **Finding your way around:** the code is commented with the reasoning as well as the
  mechanics, and these are good places to start:
  - In `packages/release-deploy`: `lib/registry.mjs` for the release lifecycle,
    `lib/branch.mjs` for how a working copy stays isolated from the trunk, `lib/merge.mjs` for
    conflict resolution, and `lib/deploy.mjs` for the upsert-by-key.
  - In `packages/console`: `lib/product-edit.ts` for fork-on-first-edit, and
    `lib/predicate/parse.ts` for the round-trip guard on discount predicates.
  - [docs/architecture.md](docs/architecture.md) ties them together.
  - `packages/release-deploy/test/` covers the branch registry under concurrent writes,
    field-level merge conflicts, the approval gate and the deploy/rollback round trip. It runs
    in about a second with `npm test`.
  - [FEATURES.md](FEATURES.md) is a code-derived inventory of what is implemented, and the
    quickest answer to "does it do X".
- **Questions and bugs:** open a GitHub issue. Another reader or a maintainer may well have an
  answer, though we can't commit to one.
- **Using it in production:** feel free to fork it and adapt it to your needs. Once it's in
  production it's yours, so review the code, test it against your own projects and keep your
  copy up to date.
- **commercetools products:** for questions about Composable Commerce or the Merchant Center
  themselves, the [commercetools documentation](https://docs.commercetools.com), the community
  and commercetools Support are the right place, as usual.
