---
name: jj-vcs
description: >
  Use Jujutsu (jj) for version-control work when the user requests a jj workflow or
  the target repository is jj-managed. Determine the actual target repository and
  its policies before acting; do not select jj from a parent directory alone.
---

# Jujutsu workflow

Use this skill only for the repository that owns the requested change. First identify
that target directory and inspect its repository policy. A `.jj` directory is evidence
about that directory, not a reason to run version-control commands for unrelated work.
If the target is a Git repository, follow its Git workflow instead. A workspace may
contain several repositories; never operate on the parent when the requested target is
a subrepository.

## Safety and policy

- Read the target repository's contribution, signoff, remote, base, rewrite, and
  workspace policies before mutating history. Follow those policies over this skill.
- Treat existing work as valuable. Inspect `jj status`, `jj log`, and `jj diff` before
  changing it, and preserve unrelated changes.
- Protect published changes. A pushed change is not automatically immutable, but do
  not rewrite a pushed or shared change unless the user gives exact authorization for
  that rewrite and the repository policy permits it. Do not use an unverified force
  workflow.
- `jj undo`, `jj operation restore`, and `jj abandon` mutate repository state. Scope
  them to an explicitly inspected operation or revision; they are recovery or cleanup
  tools, not universally safe escape hatches. Inspect `jj op log` first when recovering.
- Do not delete remotes or bookmarks as routine cleanup. Delete or forget one only
  when the user explicitly requests that exact name and the repository policy allows it.
- Do not invent force options. Check the installed `jj` help when a command or
  remote-revision form is uncertain.

## Inspect and describe

The working copy is a change and file edits are tracked without a staging area. Use
these read-oriented commands before and after a mutation:

```bash
jj status
jj log -r <revset>
jj diff
jj diff -r <revision>
jj op log
```

`@` names the working-copy revision. Change IDs can be abbreviated only when the
prefix is unambiguous. Use revsets deliberately and quote expressions containing
shell metacharacters.

## Make changes

Use an existing change when appropriate, or create a new one from the selected base:

```bash
jj new <base-revision>
jj describe -m "<change-description>"
```

`jj new` with multiple parent revisions creates a merge change; do that only when the
requested history explicitly needs a merge. Do not create speculative skeleton
changes. `jj commit -m "<change-description>"` describes the working copy and then
creates a new change, so inspect the resulting working-copy location rather than
assuming you remain on the described revision.

For focused history edits, use exact revisions and review the resulting graph:

```bash
jj edit <revision>
jj split -r <revision>
jj squash --from <revision> --into <destination>
jj rebase --source <source> --onto <selected-base>
```

These commands rewrite or relocate changes. Confirm the scope, check immutable and
published boundaries, and inspect `jj status`, `jj log`, and `jj diff` afterward.
Do not automatically rebase onto a conventional branch. Fetch and select the base
required by the target repository and task.

If the repository requires signoff, add the required trailer to the change description
using the repository's documented identity and format. Do not guess an identity.

## Bookmarks and remotes

Bookmarks are local names for revisions that matter to collaboration; local work does
not require one. Remote commands require an explicit remote and an explicit bookmark
or revision. Replace angle-bracket placeholders with values inspected from the target
repository; placeholders are syntax, not commit messages:

```bash
jj git fetch --remote <remote> --branch <bookmark>
jj log -r '<base-bookmark>@<remote>'
jj bookmark create <bookmark> --revision <revision>
jj bookmark set <bookmark> --revision <revision>
jj git push --remote <remote> --bookmark <bookmark>
```

Use `jj git fetch --remote <remote> --branch <bookmark>` before relying on a
remote-tracking revision. `<base-bookmark>@<remote>` is the remote-bookmark
revision form. The normal push safety checks protect against a stale remote.

Remote deletion is an explicit destructive operation. Use only the target
repository's documented exact-name workflow after authorization; never issue a
blanket deletion or push all deleted bookmarks.

Never use a blanket push, remote deletion, or guessed force option. If a requested
rewrite would require changing a published remote bookmark, stop at the safety boundary
and obtain exact authorization and repository-specific instructions.

## Integration and workspaces

Choose integration deliberately. Rebase onto the selected remote or local base only
when the task or repository policy calls for it; do not make upstream rebasing an
automatic step. Merge changes only when the task calls for a merge and the repository
permits it. For parallel working copies, inspect the repository's workspace policy and
then use the documented `jj workspace add <path>` workflow; do not assume a workspace
is safe merely because the command exists.

After a coherent history-changing sequence, inspect the affected revisions and diff,
confirm the intended bookmark/revision targets, and report any conflicts or rewritten
descendants before pushing. Keep remote actions separate from local history edits so
each operation is reviewable.
