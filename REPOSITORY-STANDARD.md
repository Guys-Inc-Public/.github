# Guys Inc repository standard

Every public Guys Inc repository is set up the same way, so that a contributor
who has worked in one has already learned the others.

[`archivist`](https://github.com/Guys-Inc-Public/archivist) is the reference
implementation. When this document and that repository disagree, the repository
is right and this document needs a pull request.

---

## 1. Inherited, not copied

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
| `README.md` | What it is in two sentences, then a working example. Badges for CI and licence. |
| `LICENSE` | MIT, unless there is a stated reason otherwise. |
| `CHANGELOG.md` | [Keep a Changelog](https://keepachangelog.com/) format, semantic versioning. |
| `.github/CODEOWNERS` | Review is required from code owners, so this file is what makes that meaningful. |
| `.github/dependabot.yml` | The language ecosystem plus `github-actions`. Grouped updates. |
| `.github/labels.yml` | The standard label set, applied by `sync-labels.yml`. |
| `.github/release.yml` | Categories for generated release notes. |
| `docs/` | Documentation source of truth. See §4. |
| `docs/decisions/` | ADRs for anything expensive to reverse. |

## 3. Repository settings

Set on creation. `archivist` is the worked example of all of them.

- **Merge strategy:** squash only. Merge commits and rebase merging are off.
  The PR title becomes the commit subject, so PR titles are written in the
  imperative.
- **Delete head branches on merge:** on.
- **Allow auto-merge:** on.
- **Wiki:** on, and generated — never authored. See §4.
- **Discussions:** on. Questions and ideas go there; issues are for reproducible
  defects.
- **Projects:** off unless the repository actually uses a board.
- **Security:** secret scanning, push protection, non-provider patterns,
  Dependabot alerts, Dependabot security updates, and code scanning all on.
  The organisation enables the first set by default on new repositories; code
  scanning is enabled per repository.
- **Topics:** at least five, so the repository is findable.
- **Description and homepage:** always set. A repository with no description
  reads as abandoned.

### Rulesets

Branch and tag protection is enforced by **organisation-level rulesets**, not
per-repository settings, so a new repository is protected the moment it exists.

| Ruleset | Applies to | Enforces |
|---|---|---|
| *Default branch protection* | every repository, default branch | Pull request required, 1 approving review, stale reviews dismissed, code-owner review, last-push approval, review threads resolved, squash-only, no force push, no deletion |
| *Release tags are immutable* | every repository, `v*` and `release-*` | No deletion, no updating, no force push |

Organisation admins can bypass, so an owner is never locked out of their own
repository. Nobody else can.

Required status checks are **per repository**, because check names differ. Add a
repository ruleset naming them.

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

Copied from `archivist` and adjusted for the language.

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
- Pin any action with no maintained floating major tag to an exact version.
- Every job that writes to a shared destination declares a `concurrency` group.
- Never end a step in `|| true`. A swallowed failure in a publish step produces
  a green run and a broken artefact, which is worse than a red run.

## 6. Releases

- Tags are `vMAJOR.MINOR.PATCH` and are immutable.
- Every release is signed, ships an SBOM, and records a build-provenance
  attestation.
- Signing keys: CI holds a **signing subkey**; the primary key is
  certify-only and stays offline. Rotation is documented before launch, not
  after the first expiry. See
  [archivist's signing key documentation](https://github.com/Guys-Inc-Public/archivist/blob/main/docs/Signing-Keys.md).

## 7. Checklist for a new repository

- [ ] Description, homepage, and at least five topics set
- [ ] MIT `LICENSE`, `README.md`, `CHANGELOG.md`
- [ ] Squash-only merges, delete branch on merge, auto-merge allowed
- [ ] Discussions on, Projects off, Wiki on
- [ ] Secret scanning, push protection, Dependabot, code scanning on
- [ ] `.github/CODEOWNERS`, `dependabot.yml`, `labels.yml`, `release.yml`
- [ ] Workflows copied from `archivist` and adjusted for the language
- [ ] Repository ruleset naming the required status checks
- [ ] Wiki opened once so its remote exists
- [ ] `docs/Home.md` and `docs/decisions/README.md` created
- [ ] First release tagged only after CI has been green on the default branch
