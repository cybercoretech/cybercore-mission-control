# Cybercore organization migration plan

## Objective

Move the Cybercore family of projects from the personal GitHub account into the
[`cybercore-tech`](https://github.com/cybercoretech) organization while preserving
history, CI, deployments, credentials, operator access, and rollback capability.

This is a staged ownership migration, not a single bulk transfer. No repository
ownership, Cloudflare resource, DNS record, or production deployment changes are
authorized by this document alone.

## Cybercore Tech front page

The organization should have a Cybercore-specific landing experience based on the
existing `darkstardevx` profile design, but with organization-owned names, links, and
assets. The two repositories do not exist yet and should be created during organization
readiness:

- `cybercoretech/cybercore-tech` — public organization profile repository. Its root
  `README.md` becomes the front page shown at `github.com/cybercoretech`.
- `cybercoretech/cybercoretech.github.io` — optional public GitHub Pages site for a
  richer, fully interactive landing page at `cybercoretech.github.io`.

### Recommended presentation model

Use both surfaces with a shared visual system:

- The organization profile README is the concise GitHub-native entry point.
- The Pages site is the richer interactive version with responsive layout, inline SVG,
  project filtering, and more detailed navigation.
- Both surfaces use Cybercore Tech branding and link to organization repositories first.
- The profile README links to the Pages site when the site is ready.

### Clickable SVG rule

GitHub renders SVGs referenced through `<img>` as images; links embedded inside the SVG
are not a reliable clickable surface there. The existing profile uses the correct pattern:
wrap each tile image in an HTML anchor.

```html
<a href="https://github.com/cybercoretech/cybercore-mission-control">
  <img src="./assets/tile-mission-control.svg" alt="Mission Control">
</a>
```

Decorative headers can remain plain images. Every project tile must have its own external
`<a>` wrapper, accessible `alt` text, and an organization URL. On the GitHub Pages site,
inline SVG anchors may be used for richer interactions, but the README should continue to
use the wrapper pattern for reliable GitHub rendering.

### Front-page acceptance criteria

- The profile repository is public and named exactly `cybercore-tech`.
- The README renders correctly on the organization profile without broken assets.
- Every project tile points to the correct `cybercore-tech/<repository>` URL.
- No personal-account links remain except intentionally labeled archive/mirror links.
- SVGs contain no credentials, host-specific tokens, private paths, or telemetry.
- The Pages site, if enabled, has a working link from the README and passes keyboard and
  mobile navigation checks.
- The visual system is shared through checked-in assets rather than manually duplicated
  screenshots.

## Guiding principles

1. Inventory before changing anything.
2. Migrate one repository at a time.
3. Keep the current personal repositories and deployment resources as the source of
   truth until the destination has passed its verification gate.
4. Recreate and verify secrets by name and scope; never copy secret values into chat,
   issues, commits, or logs.
5. Keep GitHub repository ownership, Cloudflare resource ownership, and local operator
   credentials as separate migration decisions.
6. Migrate Mission Control last because it is deployment-critical and currently backs
   the staging observation plane.

## Repository redundancy and synchronization policy

Both namespaces can remain available, but they should not be independently writable.
The recommended model for each migrated repository is:

```text
cybercore-tech/project  (canonical, writable)
          |
          +--> darkstardevx/project  (read-only mirror / recovery copy)
          |
          +--> local bare mirror     (offline recovery copy)
```

### Canonical repository

- `cybercore-tech/project` is the only repository where normal development occurs.
- Pull requests, issues, releases, Actions workflows, and deployment decisions belong
  to the organization repository.
- Its default branch and tags are protected.

### Personal-account mirror

- Keep `darkstardevx/project` as a separate repository when a second live copy is desired.
- Synchronize branches and tags from the organization repository after accepted changes
  and releases.
- Protect the mirror's default branch and do not accept independent feature work there.
- Mark the repository README as a mirror and direct contributors to the organization URL.
- Keep mirror Actions disabled or limited to backup/verification tasks so it cannot deploy
  or create a second source of truth.

### Synchronization method

Use one-way synchronization from the organization repository. A narrowly scoped GitHub
App installation token is preferred; a fine-grained token limited to the single personal
mirror repository is an acceptable interim option. The sync should:

- mirror the canonical default branch and release tags;
- run after pushes to the canonical repository and on a scheduled repair interval;
- fail loudly if the mirror cannot be updated;
- never copy secrets, environment values, deployment credentials, or Actions artifacts;
- record the source commit SHA and destination result in workflow logs.

Do not implement automatic two-way synchronization. It creates competing histories,
force-push risks, conflict loops, ambiguous issue/release ownership, and a dangerous
possibility that an unreviewed personal-namespace change deploys. If work is ever made in
the personal repository, import it explicitly through a reviewed pull request or manual
cherry-pick into the canonical repository.

### Transfer versus mirror-preserving migration

If both URLs must remain active, create the organization repository and mirror the verified
history into it instead of using GitHub's repository-transfer operation. A true GitHub
transfer is simpler, but the original personal URL generally becomes a redirect rather than
an independently maintained backup. Choose the mode per repository during O1/O2 and record
the decision in the inventory.

Redundancy is not complete until the mirror has been restored into a temporary repository or
local clone and its commit, tag, and release history has been verified.

## Current state

- Organization: `https://github.com/cybercoretech`
- Migration status: organization created; inventory and sequencing pending
- Current Mission Control repository: `darkstardevx/cybercore-mission-control`
- Current staging deployment: Cloudflare Worker and D1 resources remain under the
  existing Cloudflare account
- Current CI: GitHub Actions deploys staging from the current repository after the CI
  gate and protected-environment controls
- Current local operator state: connector credentials and Cloudflare Access service
  credentials remain on the operator machine and are not repository assets

## Phase 0 — Freeze the migration boundary

Before changing repository ownership:

- Do not change Cloudflare account ownership, Worker names, D1 database IDs, DNS, or
  Access applications as part of a GitHub transfer.
- Do not rotate working production or staging credentials solely because a repository
  is moving.
- Do not commit `.env`, `.dev.vars`, connector credential files, Cloudflare service
  token values, or GitHub secret values.
- Select one migration owner and one rollback owner.
- Choose a maintenance window for each repository with no release or schema migration
  scheduled during that window.

Deliverable: a written list of repositories and infrastructure explicitly in and out of
scope for this migration.

## Phase 1 — Read-only repository inventory (O1)

Create an inventory for every Cybercore-related repository in the personal account.
For each repository record:

- current URL, visibility, default branch, current head SHA, and last release;
- purpose and dependency relationships;
- collaborators, teams, outside collaborators, deploy keys, webhooks, and GitHub Apps;
- branch protection, rulesets, required checks, environments, reviewers, and wait timers;
- workflow files, reusable workflows, permissions, concurrency, and deployment targets;
- Actions variables and secret names, without recording secret values;
- package registries, release assets, Pages sites, container images, and external links;
- Cloudflare Workers, D1 databases, Access applications, tunnels, DNS, and other
  resources referenced by the repository;
- local clones, remotes, release automation, and rollback contacts.

Recommended read-only checks for each local clone:

```sh
git remote -v
git fetch --all --prune
git rev-parse HEAD
git status --short
git fsck --full
```

Recommended GitHub inventory fields:

```text
repository | visibility | default branch | head SHA | workflows | environments |
secret names | variables | deploy target | packages | webhooks | owner | risk | order
```

Deliverable: `cybercore-tech` migration inventory reviewed by the migration owner.

## Phase 2 — Prepare the organization (O2)

Configure the organization before transferring repositories:

- Confirm at least two organization owners and enable strong account security.
- Create teams based on responsibilities, not individual repositories; use least
  privilege for write, maintain, triage, and admin access.
- Set repository visibility and forking defaults deliberately.
- Define who may create repositories, Actions workflows, environments, deploy keys,
  and organization secrets.
- Decide whether Actions are allowed to run from forks and whether approval is required
  for first-time contributors.
- Establish naming, issue labels, release, security reporting, and branch conventions.
- Decide whether packages belong in the organization and which registries are supported.
- Define the organization-level secret policy and the owner for each deployment secret.
- Verify that required third-party GitHub Apps can be installed for the organization.

Do not move Cloudflare ownership in this phase. The organization can own source code
while the current Cloudflare account continues to own staging infrastructure.

Deliverable: organization baseline approved and team access tested with a non-critical
repository.

## Phase 3 — Classify and sequence repositories (O1/O2)

Use four risk classes:

| Class | Repository type | Migration order |
| --- | --- | --- |
| A | Documentation, experiments, or non-deploying utilities | First |
| B | Libraries, plugins, and local tools with release workflows | Early |
| C | AgentForge or other shared orchestration projects | Middle |
| D | Mission Control and repositories coupled to Cloudflare or live staging | Last |

For each repository, identify whether it is:

- transferred to `cybercore-tech`;
- mirrored into the organization while remaining personal temporarily;
- archived or consolidated;
- intentionally retained in the personal account.

The first pilot should be a Class A or low-risk Class B repository with no production
deployment and no irreplaceable external integration.

Deliverable: ordered migration queue with an owner, risk level, maintenance window,
verification checklist, and rollback owner per repository.

## Phase 4 — Prepare each repository (O3)

Complete this checklist before transferring a repository:

- Ensure the working tree is clean and the current head SHA is recorded.
- Push or preserve all intended branches and tags.
- Remove accidental secrets and rotate any credential that was ever committed.
- Replace hard-coded personal namespace references in documentation, badges, workflow
  permissions, release URLs, and dependency declarations where appropriate.
- Confirm workflow permissions use the minimum required access.
- Export the names of Actions secrets and variables, environment names, protection rules,
  required reviewers, and required checks.
- Confirm webhooks, deploy keys, GitHub Apps, package permissions, and release automation.
- Record deployment URLs, resource identifiers, and the last known-good workflow run.
- Capture a repository archive or verified clone for recovery.

Do not replace secrets with temporary values just to make the transfer pass. If a secret
cannot be recreated safely, stop that repository and resolve ownership first.

Deliverable: transfer-ready repository checklist signed off by the repository owner.

## Phase 5 — Pilot migration (O4)

Migrate the first low-risk repository using the mirror-preserving mode and validate the
complete lifecycle:

1. Create the destination repository under `cybercore-tech` without deleting or transferring
   the personal source.
2. Verify default branch, commit history, tags, releases, issues, discussions, and
   visibility.
3. Perform a final source freeze, mirror the verified branches and tags, and record the
   source and destination head SHAs.
4. Configure the one-way organization-to-personal mirror and verify a test update.
5. Recreate or validate branch protections, rulesets, environments, variables, secrets,
   webhooks, deploy keys, and GitHub App access.
6. Run the full CI workflow from the destination repository.
7. Exercise the repository's release or staging workflow if one exists.
8. Update documentation and dependent repositories to the organization URL.
9. Restore the mirror into a temporary location and compare commit/tag history.
10. Record the destination URL, commit SHA, workflow run, artifacts, mirror result, and
    any deviations.

Exit gate: the pilot has green CI, working required integrations, correct access, and a
documented rollback path before the next repository is scheduled.

## Phase 6 — Repeat staged migrations (O4)

For each remaining repository:

- announce the window and freeze releases for the duration;
- run the preparation checklist and record the source SHA;
- create or transfer only that repository according to its recorded migration mode;
- recreate/verify Actions environments and secret names;
- run CI and a repository-specific smoke test;
- update cross-repository references and local remotes;
- record evidence and obtain sign-off;
- wait for the agreed observation period before migrating the next repository.

For mirror-preserving migrations, the personal repository remains available as a recovery
copy. For true GitHub transfers, preserve an independent local or offline mirror before
accepting the transfer because the old personal URL may become only a redirect.

Do not combine multiple repository transfers with dependency upgrades, schema migrations,
Cloudflare changes, or broad credential rotation. Separate changes make failures
diagnosable and rollback practical.

## Phase 7 — Mission Control final migration gate (O5)

Mission Control moves only after the previous phases are complete and staging is healthy.

### Pre-transfer requirements

- Current `main` CI is green and the release/deployment commit is recorded.
- The staging dashboard, API, live event stream, agent registration, heartbeat, stale
  detection, and audit history have all been verified.
- The current Worker name, custom hostname, D1 database, Durable Object binding, and
  Access application are documented.
- The current GitHub `staging` environment has a complete secret/variable name inventory:
  - `CLOUDFLARE_ACCOUNT_ID` variable;
  - `CLOUDFLARE_API_TOKEN` secret;
  - `ADMIN_TOKEN` secret.
- The Cloudflare Access policy, service-token policy, and connector credential ownership
  are recorded without exposing values.
- A rollback owner can restore the previous known-good workflow and repository remote.

### Transfer and rewire

- Transfer `darkstardevx/cybercore-mission-control` to `cybercore-tech`.
- Confirm the default branch, history, tags, workflows, issues, and repository visibility.
- Recreate or validate the `staging` environment and its protection settings.
- Re-add the required secret and variable names from the inventory; do not paste values
  into the repository or chat.
- Verify the Cloudflare API token is permitted to deploy the existing Worker/resources.
- Update local remotes and repository links.
- Run CI from the organization repository and wait for the automatic staging deployment.
- Verify the staging URL, dashboard authentication, API authorization, event stream,
  registered agent, heartbeat, online count, stale state, and recent audit event.
- Confirm no D1 migration ran unintentionally during deployment.

### Mission Control exit gate

The transfer is accepted only when the organization repository has a green CI run, a
successful staging deployment, a successful connector heartbeat, and recorded evidence
for the checks above. If any gate fails, stop further transfers and restore the last
verified source/deployment configuration.

## Phase 8 — Post-migration cleanup

After the observation period for all selected repositories:

- update README badges, clone URLs, release links, documentation, and dependency references;
- update local clones and remove stale personal remotes only after verification;
- review organization audit logs, Actions history, deploy keys, webhooks, and installed Apps;
- remove temporary transfer access and obsolete credentials;
- archive or mark the personal repositories as redirected/migrated according to the chosen
  policy; do not delete the source until retention requirements are satisfied;
- update the ownership map, on-call contacts, secret rotation owners, and rollback records;
- publish a short migration report with repository URLs, commit SHAs, workflow evidence,
  deployment evidence, and unresolved follow-up work.

## Cloudflare and credential boundary

GitHub repository transfer does not automatically transfer Cloudflare resources. Keep these
as separate workstreams:

- GitHub organization ownership: source, workflows, issues, releases, and Actions settings.
- Cloudflare account ownership: Workers, D1, Durable Objects, Access, DNS, and API tokens.
- Operator machine: local connector credential and Cloudflare Access service-token files.

For the current staging deployment, the safest first migration is to move source ownership
while continuing to deploy the existing Cloudflare resources with a narrowly scoped token.
Cloudflare account migration, if desired later, should receive its own plan and approval.

## Risks and mitigations

| Risk | Mitigation |
| --- | --- |
| Actions secrets or environment rules are missing after transfer | Inventory names first; recreate and run staging CI before accepting the transfer |
| Cloudflare token cannot deploy from the new repository | Verify token scope and staging workflow before the Mission Control transfer |
| Hidden webhook, App, or deploy-key dependency breaks | Inventory integrations and run a post-transfer event/deploy check |
| Personal namespace remains in docs or automation | Search source, workflows, releases, and dependent repositories before sign-off |
| Transfer is combined with unrelated code or infra changes | Freeze the migration boundary and use one repository/change class per window |
| A credential is exposed during migration | Never place values in chat or commits; revoke and rotate immediately if exposed |
| Rollback is unclear | Record source SHA, workflow run, destination state, and named rollback owner |

## Definition of complete

The migration is complete only when every selected repository has:

- a verified `cybercore-tech` URL and preserved history;
- working CI, required environments, integrations, and release automation;
- documented secret ownership and recreated secret names;
- verified staging/production behavior where applicable;
- updated cross-repository references and local remotes;
- a tested or explicitly accepted rollback path;
- an owner for ongoing maintenance and credential rotation.

Until those conditions are met, the personal repositories remain the operational source of
truth.

## Immediate next action

Start O1 with a read-only inventory of Cybercore repositories and integrations. Do not transfer
repositories, rotate credentials, or change Cloudflare ownership during the inventory.
