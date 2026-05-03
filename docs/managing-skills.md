# Managing Skills

This repository is intentionally small. A skill is just a folder with a
`SKILL.md` file inside it:

```text
skills/
  my-skill/
    SKILL.md
```

The `SKILL.md` file needs YAML frontmatter with a `name` and `description`.
That frontmatter is how agents decide when the skill should be loaded, so treat
it like the skill's public API.

```markdown
---
name: my-skill
description: Use when...
---

# My Skill

Instructions for the agent.
```

## Create A Skill

Create a folder under `skills/`:

```bash
mkdir -p skills/my-skill
npx skills@latest init skills/my-skill
```

Edit `skills/my-skill/SKILL.md` until the skill is clear and compact.

Use these rules of thumb:

- Keep the skill name in kebab-case.
- Make the description specific about when the skill should be used.
- Put only agent-facing instructions in `SKILL.md`.
- Add bundled files only when they help the agent do the work: `scripts/`,
  `references/`, or `assets/`.
- Keep human-facing process docs in this repository's `docs/` folder, not inside
  the skill folder.

Register the skill in `.claude-plugin/plugin.json`:

```json
{
  "name": "gvergnaud-skills",
  "skills": ["./skills/typescript-code-quality", "./skills/my-skill"]
}
```

Add the skill to the README:

```markdown
- [my-skill](./skills/my-skill/SKILL.md) - One sentence explaining its purpose.
```

Validate discovery before publishing:

```bash
npx skills@latest add . --list
```

## Update A Skill

Edit the skill:

```bash
$EDITOR skills/typescript-code-quality/SKILL.md
```

If you change the skill's purpose, update the `description` frontmatter too.
That description is not decoration; it is the trigger surface.

If you add, rename, or remove bundled files, make sure `SKILL.md` explains when
the agent should read or use them.

Run the local discovery check:

```bash
npx skills@latest add . --list
```

Then review and publish the change:

```bash
git diff
git add .
git commit -m "Update typescript-code-quality skill"
git push
```

## Delete A Skill

Remove the skill folder:

```bash
rm -rf skills/my-skill
```

Remove the skill path from `.claude-plugin/plugin.json`.

Remove the skill from the README.

Check that the repository still exposes the expected skills:

```bash
npx skills@latest add . --list
```

Commit and push:

```bash
git add .
git commit -m "Remove my-skill skill"
git push
```

## Rename A Skill

Renaming is a delete plus a create. Do it deliberately, because users who have
installed the old skill name will not automatically know about the new one.

Move the folder:

```bash
mv skills/old-name skills/new-name
```

Update the `name` field in `skills/new-name/SKILL.md`.

Update `.claude-plugin/plugin.json` and the README.

Validate:

```bash
npx skills@latest add . --list
```

Commit and push:

```bash
git add .
git commit -m "Rename old-name skill to new-name"
git push
```

## Publish Changes

Publishing a skill update means pushing to GitHub:

```bash
git push
```

Users can then update their installed copy:

```bash
npx skills@latest update
```

They can also reinstall from the repository if they want a fresh copy:

```bash
npx skills@latest add gvergnaud/skills --skill typescript-code-quality
```

## Make The Repository Public

The repository is private while the skill list is still shaping up. When it is
ready, make it public:

```bash
gh repo edit gvergnaud/skills \
  --visibility public \
  --accept-visibility-change-consequences
```

Before making it public, quickly review the repository for private notes,
temporary files, and GitHub Actions logs you would rather not expose.
