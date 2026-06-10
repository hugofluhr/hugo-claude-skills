# hugo-claude-skills

Personal library of Claude Code skills — reusable knowledge modules that load into Claude Code conversations via the FleetView skill system.

## What's a skill?

Skills are markdown files that inject focused domain knowledge into Claude Code. When a trigger condition is matched (e.g., you mention "the cluster" or a file imports `anthropic`), the skill is loaded automatically, giving Claude the context it needs without you having to re-explain it every session.

## Skills

| Skill | Description | Source |
|---|---|---|
| [`sciencecluster`](skills/sciencecluster/SKILL.md) | SLURM cluster (UZH sciencecluster) operational knowledge — job submission, conda in SLURM, GPU constraints, log conventions, and common failure modes | Adapted from [Gilles de Hollander](https://github.com/Gilles86/gilles-claude-skills/tree/main) |

## Structure

```
skills/
  <skill-name>/
    SKILL.md          # Main skill file with frontmatter (name, description, triggers)
    references/       # Optional: supplementary reference files loaded on demand
```
