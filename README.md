# flutter-skills

A public repository of Flutter-focused Codex skills for refactoring, code review, and engineering quality checks.

## What this repo contains

This repository currently publishes two focused skills for Flutter codebases:

- `memory-leak`: Reviews Flutter and Dart changes for real memory leaks and lifecycle cleanup bugs.
- `widget-split`: Refactors deeply nested Flutter widget trees into stepwise `resultWidget` wrapping and smaller private `_buildXxx()` helpers.

It is designed for Flutter maintenance and refactoring workflows, with emphasis on:

- readable widget-tree refactors that preserve behavior
- undisposed controllers and disposable objects
- retained listeners and callbacks
- `Timer` and `StreamSubscription` lifecycle leaks
- `OverlayEntry`, `TextPainter`, and image stream cleanup
- known third-party leak risks

## Repository layout

Skills live under `skills/<skill-name>/` so they can be discovered by Git-based skill tooling and repository indexes.

```text
skills/
  widget-split/
    SKILL.md
    references/
      examples.md
    agents/
      openai.yaml
  memory-leak/
    SKILL.md
    references/
      project-conventions.md
      project-wrappers.md
      examples.md
    agents/
      openai.yaml
```

## Use with the skills CLI

List the skills exposed by this repository:

```bash
npx skills add zeqinjie/flutter-skills --list
```

Install only the `memory-leak` skill:

```bash
npx skills add zeqinjie/flutter-skills --skill memory-leak
```

Install only the `widget-split` skill:

```bash
npx skills add zeqinjie/flutter-skills --skill widget-split
```

You can also reference the repository by GitHub URL if needed:

```text
https://github.com/zeqinjie/flutter-skills
```

## skills.sh indexing

This repository is structured to be compatible with `skills.sh` discovery:

- `skills/memory-leak/SKILL.md`
- `skills/memory-leak/agents/openai.yaml`
- `skills/widget-split/SKILL.md`
- `skills/widget-split/agents/openai.yaml`
- `skills.sh.json`

For `skills.sh`, publishing is not just about pushing a repository. The repository must also be seen by the `skills` ecosystem, typically when someone runs the CLI against it.

In practice, the usual flow is:

1. Push the repository to a public GitHub repo.
2. Run `npx skills add zeqinjie/flutter-skills --list` to verify discovery.
3. Run `npx skills add zeqinjie/flutter-skills --skill memory-leak` or `npx skills add zeqinjie/flutter-skills --skill widget-split` to install a skill.
4. Wait for `skills.sh` indexing and cache refresh.

## Main skill file

The published skill definition is here:

- [skills/widget-split/SKILL.md](skills/widget-split/SKILL.md)
- [skills/memory-leak/SKILL.md](skills/memory-leak/SKILL.md)

Reference files loaded on demand:

- [skills/widget-split/references/examples.md](skills/widget-split/references/examples.md) - official Flutter before/after refactor patterns
- [skills/memory-leak/references/project-conventions.md](skills/memory-leak/references/project-conventions.md) — optional project-specific rules
- [skills/memory-leak/references/project-wrappers.md](skills/memory-leak/references/project-wrappers.md) — local wrapper implementations
- [skills/memory-leak/references/examples.md](skills/memory-leak/references/examples.md) — true-positive and false-positive patterns
