# Workspaces

A workspace is another working copy of the same repository, like
`git worktree`: its own directory and `@`, sharing changes, bookmarks, and the
operation log. Follow the repository's workspace policy first.

```bash
jj workspace add --name <name> -r <base> <path>
jj workspace list
jj workspace forget <name>   # stop tracking; files stay on disk
jj workspace remove <name>   # snapshot, then delete the directory
```

- `jj log` shows each workspace's `@` as `<name>@`. Other workspaces' changes
  appear at the next jj command; files never sync live.
- When another workspace rewrites this one's `@`, commands here fail as stale
  until `jj workspace update-stale`.
- Give each workspace its own `@`. If two share one through `jj edit`, a
  rewrite in one turns the other's unsnapshotted edits into a divergent change
  (`<id>??` in `jj log`, revset `divergent()`) that must be resolved.
- Retire a workspace with `forget` or `remove`; deleting the directory alone
  leaves the workspace and its `@` tracked in the repository.
