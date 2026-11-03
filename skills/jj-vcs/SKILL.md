---
name: jj-vcs
description: >
  Jujutsu (jj) version control. Use when the target repo is jj-managed or the
  user asks for jj, including before git commands in a `.jj` repo (its
  detached HEAD is normal). Check the target repo and its policies first; a
  parent's `.jj` does not count.
---

# Jujutsu workflow

Written for jj 0.46; check `jj help <command>` rather than guessing flags.

## Target and policy

Act only in the repository that owns the requested change. A `.jj` directory
describes that directory alone: in a workspace of several repositories, operate
on the subrepository the task names, never its parent. A plain Git target keeps
its Git workflow.

Read the target's contribution, signoff, remote, base, rewrite, and workspace
policies before mutating history; they override this skill. Inspect `jj st`,
`jj log`, and `jj diff` before and after each mutation, and preserve unrelated
changes. Add a required signoff trailer with the repository's documented
identity, never a guessed one.

## Agent hazards

Editor and diff-tool forms hang or fail without a terminal; use the agent form.

| Opens an editor or TUI | Agent form |
|---|---|
| `jj describe`, `jj commit` | add `-m "<msg>"` |
| `jj split` | `jj split -m "<msg>" <paths>` |
| `jj squash` when both sides are described | add `-m "<msg>"`, or `-u` to keep the destination's |
| `jj squash -i`, `jj resolve` | pass paths; edit conflict markers in the files |

- `jj diff --git` prints a unified diff; the default format is easy to misread.
- Every jj command snapshots the working copy, so new files land in `@` at once.
  Add build output and secrets to `.gitignore`, then `jj file untrack <paths>`.
- In a colocated repo (`.jj` beside `.git`), git is read-only and a detached
  HEAD is the normal state; leave it detached.
- `<placeholders>` are syntax: substitute values inspected from the repository.

## Change and refine

| Intent | Command |
|---|---|
| Describe `@` and start its child | `jj commit -m "<msg>"` (`@` moves to the child) |
| Fold changes into a change | `jj squash --from <rev> --into <dest> [paths]` |
| Fold lines into the ancestors that last touched them | `jj absorb` |
| Split paths into a new first change | `jj split -r <rev> -m "<msg>" <paths>` |
| Move a change and its descendants | `jj rebase -s <rev> -o <base>` (`-r`: without descendants; `-o`, not `-d`) |
| Discard or copy file content | `jj restore [--from <rev>] <paths>` |
| Reverse a change with a new one | `jj revert -r <rev> -B @` (`backout` is gone) |

Create a change when it will hold work: an empty described change is clutter
that is easy to push or leave behind. Create a merge (`jj new` with several
parents) only when the requested history needs one. Rebase only onto the base
the task or policy selects; upstream rebasing is never an automatic step.

Rewrites carry descendants along. Before one, confirm the target is mutable and
unpublished; afterward, review the rewritten descendants.

## Conflicts

jj records conflicts in changes instead of stopping, and a rewrite can conflict
its descendants. List them with `jj log -r 'conflicts()'`. Resolve each one in
place (`jj new <rev>`, edit the markers, `jj squash`), then confirm `jj st`
reports none.

## Publishing

Bookmarks are optional names for collaboration, and they stay put while you
create changes: set one before pushing. Each remote command names one remote
and one bookmark.

```bash
jj git fetch --remote <remote> --branch <bookmark>
jj log -r '<bookmark>@<remote>'
jj bookmark set <bookmark> -r <rev>
jj git push --remote <remote> --bookmark <bookmark>
jj git push --remote <remote> --named <bookmark>=<rev>   # new bookmark
```

- `jj git push` force-updates a rewritten bookmark without any flag, guarded
  only by a stale-remote check. Rewrite or re-push a published change only with
  the user's exact authorization and the repository's permission; otherwise
  stop at that boundary and ask.
- `jj bookmark delete` propagates the deletion on the next push; `forget` stays
  local. Delete a bookmark or remote only by exact name, on explicit request.
  Push named bookmarks, never `--all`, `--tracked`, or `--deleted`.
- Push in its own step after local edits. First confirm each bookmark's target
  and report conflicts or rewritten descendants.

## Recovery and workspaces

`jj undo`, `jj op restore <op>`, and `jj abandon <rev>` mutate state too. Read
`jj op log` first and scope each to an inspected operation or revision.
`jj evolog -r <rev>` shows how a change evolved.

Before adding, editing in, or removing a workspace, read
[references/workspaces.md](references/workspaces.md).
