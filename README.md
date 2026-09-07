# agent-skills

This repository is the source tree for skills managed by `npx skills`. Installable
skills live at `skills/<name>/SKILL.md`; for example, the source copy of `jj-vcs`
is `skills/jj-vcs/SKILL.md`.

Discover skills in this local source tree:

```bash
npx --yes skills add <path-to-agent-skills> --list
```

Install a local source skill globally:

```bash
npx --yes skills add <path-to-agent-skills> --skill jj-vcs --global --yes
```

On this machine, the OMP-managed `~/.pi` location is a symlink to `.omp`, so install
the skill for that agent alias with:

```bash
npx --yes skills add ~/dev/rfns/agent-skills --skill jj-vcs --global --agent pi --yes
```

`pi` is this machine's alias for OMP; do not assume that every OMP installation uses
the `pi` agent name.

The active `work` profile uses a separate skills directory. After the install,
create this link once if it does not exist:

```bash
ln -s ~/.omp/agent/skills/jj-vcs ~/.omp/profiles/work/agent/skills/jj-vcs
```

For another profile, use its `$PI_CODING_AGENT_DIR/skills/` directory instead.
The profile link follows subsequent `npx skills` reinstalls. Start a new OMP
session to load the updated catalog. The CLI does not manage OMP profile links.

List globally installed skills:

```bash
npx --yes skills list --global
```

Files in this repository are source copies. The `npx skills` installation creates or
updates installed copies in the selected agent's global skill directories; those
copies are managed artifacts, not editing locations. Change the source here, then
reinstall or update it with `npx skills` when you want installed copies refreshed.