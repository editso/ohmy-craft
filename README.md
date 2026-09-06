# ohmy-craft

A curated collection of practical skills for thoughtful Vibe Coding.

## Install all skills

```bash
npx skills add editso/ohmy-craft --all
```

Install globally:

```bash
npx skills add editso/ohmy-craft --all --global
```

Install one skill:

```bash
npx skills add editso/ohmy-craft --skill working-efficiently
```

List available skills before installing:

```bash
npx skills add editso/ohmy-craft --list
```

## Layout

Each directory under `skills/` is an independently installable skill:

```text
skills/
└── <skill-name>/
    └── SKILL.md
```

The repository intentionally uses the standard Agent Skills layout so it can be installed by the `skills` CLI and consumed by supported coding agents.

## Included skills

- `working-efficiently` - bounded exploration, independent claim checking, durable progress, and concise delivery.
- `brainstorming` - design-first collaboration before implementation, included from [obra/superpowers](https://github.com/obra/superpowers) under the MIT License.
- `codebase-design` - shared vocabulary and principles for designing deep modules, included from [mattpocock/skills](https://github.com/mattpocock/skills) under the MIT License.
