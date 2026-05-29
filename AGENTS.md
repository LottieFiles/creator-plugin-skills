# AGENTS.md

This repository is a skills-only repository for LottieFiles Creator Plugin development.

## Repository Structure

- `skills/creator-plugin-development` - Creator Plugin architecture, Creator API usage, scene graph manipulation, and plugin development workflow.
- `skills/creator-plugins-ui` - Plugin UI implementation with `@lottiefiles/creator-plugins-ui`.

## Skill Authoring Rules

- Every skill must live in `skills/<skill-name>/`.
- Every skill must include `SKILL.md` with YAML frontmatter containing `name` and `description`.
- Keep `SKILL.md` focused on trigger conditions and core workflow.
- Put detailed examples and longer references in `references/` and link to them from `SKILL.md`.
- Do not add package, plugin, or monorepo tooling back into this repository.

## Validation

Run this before considering changes complete:

```bash
npx skills add . --list
```

If the skill list is correct, optionally test installation into the target agent:

```bash
npx skills add . --skill '*' -a codex --copy
```
