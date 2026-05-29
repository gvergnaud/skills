# Gabriel's AI skills

Small, opinionated skills for AI agents.

The goal of this repository is simple: capture the workflows and principles I
want my agents to remember, then make them easy to install anywhere. Each skill
is a tiny package of instructions with a clear purpose. No framework, no build
step, no ceremony.

## Install

Install everything in this repository:

```bash
npx skills@latest add gvergnaud/skills
```

Install a specific skill:

```bash
npx skills@latest add gvergnaud/skills --skill typescript-code-quality
```

Install globally, so your agents can use the skill in every project:

```bash
npx skills@latest add gvergnaud/skills --skill typescript-code-quality --global
```

List the skills before installing:

```bash
npx skills@latest add gvergnaud/skills --list
```

This repository is private for now. These commands work for GitHub accounts with
access to the repository. Once the repository is public, anyone can install from
it with the same commands.

## Skills

- [code-quality](./skills/code-quality/SKILL.md) - Language-agnostic guidance
  for writing, reviewing, and refactoring code with local reasoning, low
  integration complexity, narrow APIs, honest domain models, and manageable
  cyclomatic complexity.
- [typescript-code-quality](./skills/typescript-code-quality/SKILL.md) -
  Guidance for writing and refactoring TypeScript with maintainability, type
  safety, explicit error handling, local reasoning, and strong module
  boundaries.

## Maintaining This Repo

If you want to add, update, remove, or publish a skill, see
[Managing Skills](./docs/managing-skills.md).
