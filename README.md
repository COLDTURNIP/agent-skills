# agent-skills

Reusable agent skills for coding agents. Installable skills live in
[`skills/<name>/SKILL.md`](./skills); for example,
[`jj-vcs`](./skills/jj-vcs/SKILL.md).

## Installation

Install from the public GitHub repository:

```bash
npx --yes skills add COLDTURNIP/agent-skills
```

The installer lets you choose which skills to install and which supported agents
to install them for.

To inspect the available skills first:

```bash
npx --yes skills add COLDTURNIP/agent-skills --list
```

To install a specific skill globally without prompts:

```bash
npx --yes skills add COLDTURNIP/agent-skills --skill jj-vcs --global --yes
```

Without `--global`, the installer writes editable skill files into the current
project. With `--global`, it installs the selected skills in the chosen agents'
global skill directories.

## Local development

When working from a clone, replace `COLDTURNIP/agent-skills` in the commands above
with the path to the clone. Make changes in this repository, then reinstall the
skill to refresh the installed copy.