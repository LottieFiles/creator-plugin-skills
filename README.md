# LottieFiles Creator Plugin Skills

Agent skills for building, maintaining, and instrumenting LottieFiles Creator Plugins.

## Skills

- `creator-plugin-development` - Use when creating or modifying Creator Plugins, working with the plugin sandbox/UI split, using `@lottiefiles/creator-api-types`, or manipulating the scene graph.
- `creator-plugins-ui` - Use when building plugin UIs with `@lottiefiles/creator-plugins-ui`, including component usage, theming, and common migration patterns.

## Install

List available skills from this repository:

```bash
npx skills add LottieFiles/creator-plugin-skills --list
```

Install one skill:

```bash
npx skills add LottieFiles/creator-plugin-skills --skill creator-plugin-development
```

Install all skills:

```bash
npx skills add LottieFiles/creator-plugin-skills --skill '*'
```

## Repository Layout

```text
skills/
├── creator-plugin-development/
└── creator-plugins-ui/
```

Each skill is self-contained in its own directory with a required `SKILL.md` file and optional `references/` files loaded by agents only when needed.

## Local Validation

From the repository root:

```bash
npx skills add . --list
```

To test installation into a specific agent:

```bash
npx skills add . --skill '*' -a codex --copy
npx skills add . --skill '*' -a claude-code --copy
```
