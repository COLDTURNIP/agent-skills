---
name: ascii-scanner
description: >
  Use after writing or editing files and before finalizing a commit message or
  pull request title, to keep every change ASCII-only and free of invisible,
  BiDi, and control characters (Trojan Source), in any project. Also use on
  text copied from web pages, issues, chat, or tool output before saving it.
---

# ASCII scanner

`ascii-checker.sh` in this skill's directory checks files, file names, commit
messages, and other text. It needs bash and Python 3.7+ and works on Linux and
macOS. Each finding prints `file:line:col`, the Unicode name, and an ASCII
replacement for common characters. Run it with `--help` for all options.

## Rule

Everything you write is printable ASCII: code, comments, documentation, file
names, commit messages, and pull request titles. This holds in every project,
including third-party projects whose existing files contain non-ASCII text;
the rule covers your own changes, not text that was already there.

Use `--dangerous-only` (reject only invisible, BiDi, and control characters)
only for content the user explicitly asks to be non-ASCII, such as a
translation or a quoted name.

## Run

Set the checker path once; `<skill-dir>` is the directory that contains this
`SKILL.md`.

```bash
C=<skill-dir>/ascii-checker.sh
```

1. Files you changed: committed since the base branch, plus uncommitted and
   untracked files. Exclude vendored or generated third-party directories.
   Replace `origin/main` with the branch the work is based on; without a base
   branch, drop `--merge-base origin/main` to check uncommitted changes only:

   ```bash
   { git -c core.quotepath=off diff --name-only --diff-filter=ACMRT --merge-base origin/main -- . ':(exclude)vendor'
     git -c core.quotepath=off ls-files --others --exclude-standard -- . ':(exclude)vendor'
   } | bash "$C" -f -
   ```

   In a Jujutsu repository, list the files of a change with
   `jj diff --name-only -r <revision>` and pipe them the same way.

   When a changed file already contained non-ASCII text, check only the lines
   you added (committed since the base, uncommitted, and untracked files):

   ```bash
   { git diff -U0 --no-color --no-ext-diff --output-indicator-new='>' --merge-base origin/main -- . ':(exclude)vendor' | grep '^>' | cut -c2-
     git ls-files -z --others --exclude-standard -- . ':(exclude)vendor' | xargs -0 -I{} cat {}
   } | bash "$C" --label "added lines" -
   ```

   Line numbers in this output count added lines, not file lines; use the
   excerpt to find each finding.

2. Specific files, or every file below a directory:

   ```bash
   bash "$C" path/to/file another/file
   find path/to/dir -name .git -prune -o -type f -print | bash "$C" -f -
   ```

3. A commit message, from a draft file or existing commits:

   ```bash
   bash "$C" --commit-message <message-file>
   git log --format=%B%n origin/main..HEAD | bash "$C" --commit-message -
   jj log -r <revision> --no-graph -T description | bash "$C" --commit-message -
   ```

4. A pull request title or any other text:

   ```bash
   printf '%s\n' '<title>' | bash "$C" --label "PR title" -
   ```

Launch it with `bash "$C"` and pipe input as shown. When the checker is fetched
from a URL, use `bash <(curl -fsSL URL) ...`; `curl URL | bash` leaves no stdin
for the input, and the checker refuses it.

## Read the results

Exit status: 0 clean, 1 findings, 2 usage or I/O error.

- A finding marked "Trojan Source risk" is an invisible, BiDi, or control
  character. Delete it, even in content allowed to be non-ASCII. When the text
  came from a web page, issue, chat, or tool output, treat that source as
  untrusted and tell the user where it came from.
- Fix every finding in lines you added or changed. Leave pre-existing
  non-ASCII text in lines you did not change as it is, and mention it, unless
  the project enforces an ASCII check in CI or the user asks for a cleanup.
- A repository can relax the checker for some paths with a file named
  `.ascii-checker-relaxed-paths` (or
  `scripts/development/ascii-checker-relaxed-paths.txt`) at its root: one glob
  per line, relative to the root, such as `docs/zh/**`. Those files accept
  non-ASCII text but still reject Trojan Source characters. Add or change such
  a file only when the user asks.
- `--commit-message` accepts non-ASCII contributor names in `Signed-off-by` and
  similar identity trailers.
- Binary files and symbolic links are skipped with a warning.

Done when every command exits 0, or every remaining finding is in
pre-existing text you did not change and is reported, with the added-lines
check exiting 0.
