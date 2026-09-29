# Governance

Small-project governance, written down so that it is predictable rather than
implicit.

## Who decides

Guys Inc projects are maintainer-led. The maintainers listed in a repository's
`.github/CODEOWNERS` decide what gets merged and what gets released.

There is no committee and no voting. What there is instead is a written record:
decisions that are expensive to reverse are recorded as ADRs in the repository's
`docs/decisions/`, with the reasoning that produced them. Anyone can read why a
thing is the way it is, and anyone can open a pull request arguing that the
reasoning no longer holds.

## How a change gets in

1. Open an issue or discussion first for anything substantial. A pull request
   that arrives without prior context may be asking for a direction the project
   has already decided against — and that wastes your time, not ours.
2. Open a pull request. Every change needs one; nobody pushes to a default
   branch, maintainers included.
3. One approving review from a code owner, and green checks.
4. A maintainer merges by squash.

Trivial changes still take this path. The value of a rule that has no exceptions
is that nobody has to judge whether this one qualifies.

## Changing a decision

Architecture decision records are immutable once merged. To change one, open a
pull request adding a new record that **supersedes** it and explains what
changed — new information, a changed constraint, or a consequence that turned
out worse than expected. The old record stays, marked superseded.

This matters more than it looks. Editing a decision in place erases the fact
that a different choice once seemed right, which is exactly the information
needed to judge whether the new one is better.

## Becoming a maintainer

By doing maintainer work: reviewing other people's pull requests, triaging
issues, and shipping changes over a period long enough to show it was not a
one-off. There is no application. An existing maintainer will ask.

Maintainers who have been inactive for a year are moved to emeritus, with
thanks. It is not a judgement, and it reverses on request — an inactive
reviewer in `CODEOWNERS` blocks contributors, which is a real cost.

## Releases

Maintainers cut releases. A release is an immutable `vMAJOR.MINOR.PATCH` tag on
`main`. Every release is signed, ships an SBOM, and records a build provenance
attestation; [archivist](https://github.com/Guys-Inc-Public/archivist) shows how.

## Code of Conduct

Enforcement is a maintainer responsibility. Reports go to **CJ@guysinc.org** and
are handled privately. See [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md).

## This document

Changed the same way as anything else: a pull request.
