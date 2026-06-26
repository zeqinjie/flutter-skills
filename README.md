# flutter-skills

A public repository of Flutter-focused Codex skills for code review and engineering quality checks.

## What this repo contains

This repository currently publishes one focused review skill for Flutter codebases:

- `memory-leak`: Reviews Flutter and Dart changes for real memory leaks and lifecycle cleanup bugs.

It is designed for code review and pre-merge checks, with emphasis on:

- undisposed controllers and disposable objects
- retained listeners and callbacks
- `Timer` and `StreamSubscription` lifecycle leaks
- `OverlayEntry`, `TextPainter`, and image stream cleanup
- known third-party leak risks

## Repository layout

Skills live under `skills/<skill-name>/` so they can be discovered by Git-based skill tooling and repository indexes.

```text
skills/
  memory-leak/
    SKILL.md
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

You can also reference the repository by GitHub URL if needed:

```text
https://github.com/zeqinjie/flutter-skills
```

## skills.sh indexing

This repository is structured to be compatible with `skills.sh` discovery:

- `skills/memory-leak/SKILL.md`
- `skills/memory-leak/agents/openai.yaml`
- `skills.sh.json`

For `skills.sh`, publishing is not just about pushing a repository. The repository must also be seen by the `skills` ecosystem, typically when someone runs the CLI against it.

In practice, the usual flow is:

1. Push the repository to a public GitHub repo.
2. Run `npx skills add zeqinjie/flutter-skills --list` to verify discovery.
3. Run `npx skills add zeqinjie/flutter-skills --skill memory-leak` to install the skill.
4. Wait for `skills.sh` indexing and cache refresh.

## Main skill file

The published skill definition is here:

- [skills/memory-leak/SKILL.md](skills/memory-leak/SKILL.md)
