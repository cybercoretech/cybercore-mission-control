# Cybercore organization migration plan

## Objective

Move the Cybercore family of projects from the personal GitHub account into the
`cybercore-tech` organization when each repository is ready, while preserving
history, deployment continuity, secrets, and operator access.

This is a planned workstream only. No repository ownership or deployment target
changes are authorized by this document.

## Current state

- Organization: `https://github.com/cybercore-tech`
- Migration status: organization created; repository inventory and sequencing pending
- Current Mission Control repository: `darkstardevx/cybercore-mission-control`
- Current staging deployment: Cloudflare Worker and D1 resources remain under the existing account
- Current CI: GitHub Actions deploys staging from the current repository after the protected CI gate

## Milestones

### O1 — Inventory and ownership map

- List Cybercore-related repositories in the personal account.
- Record each repository's purpose, default branch, active workflows, environments,
  deploy targets, package registries, webhooks, collaborators, and required secrets.
- Identify repositories that must remain personal or private.
- Choose a migration order, starting with low-risk repositories and ending with
  deployment-critical services.

### O2 — Organization readiness

- Confirm organization owners, repository visibility defaults, and two-factor authentication.
- Create the required teams and least-privilege repository access.
- Establish organization-level Actions policies and secret-management conventions.
- Decide whether Cloudflare resources remain in the current account or move to an
  organization-controlled account later; do not mix that decision with a repository transfer.

### O3 — Repository transfer preparation

- Update documentation, badges, workflow permissions, release URLs, and clone URLs
  that hard-code the personal namespace.
- Confirm all Actions secrets and environment variables can be recreated in the destination.
- Confirm deploy keys, webhooks, GitHub Apps, branch protections, environments,
  discussions, and package permissions.
- Prepare a rollback record containing the original repository URL and verified commit SHA.

### O4 — Staged repository transfers

- Transfer one repository at a time through GitHub's repository transfer flow.
- Preserve the default branch and verify the automatic redirect from the old URL.
- Recreate or validate Actions environments, reviewers, secrets, variables, and tokens.
- Run the repository's CI and deployment smoke checks before moving to the next repository.

### O5 — Mission Control transfer and post-transfer verification

- Transfer `cybercore-mission-control` only after its staging deployment and rollback
  evidence are current.
- Verify the Actions `staging` environment, `CLOUDFLARE_ACCOUNT_ID`,
  `CLOUDFLARE_API_TOKEN`, and `ADMIN_TOKEN` without exposing their values.
- Verify the staging Worker, D1 binding, Cloudflare Access application, custom hostname,
  connector heartbeat, live event stream, and release links.
- Update local remotes and all cross-repository references to `cybercore-tech`.
- Record the post-transfer commit, workflow run, deployment URL, and rollback owner.

## Exit criteria

The migration workstream is complete only when every selected repository has a verified
organization URL, working CI, working required deployments, recreated access controls,
documented secrets ownership, and a tested rollback path. Until then, the personal
repositories remain the source of truth.

## Immediate next action

Start O1 with a read-only repository inventory. Do not transfer repositories, rotate
credentials, or change Cloudflare ownership as part of the inventory.
