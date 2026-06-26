# flutter-skills

Public Flutter-focused Codex skills repository.

## Included skills

- `memory-leak`: Review Flutter and Dart code for high-confidence memory leak and lifecycle violations.

## Repository layout

Skills live under `skills/<skill-name>/` so they can be discovered by skill tooling and Git-based skill indexes.

```text
skills/
  memory-leak/
    SKILL.md
    agents/
      openai.yaml
```

## Install

With a Git-based skills workflow, the repository can be referenced directly and the `memory-leak` skill can be selected from it.

Example repository reference:

```text
zeqinjie/flutter-skills
```

Example skill path inside the repository:

```text
skills/memory-leak
```

## Publish notes

To make this repository visible on `skills.sh`, push this repository to GitHub and keep it publicly accessible.

This repository already includes:

- `skills/memory-leak/SKILL.md`
- `skills/memory-leak/agents/openai.yaml`
- `skills.sh.json`

After pushing, use the repository URL or owner/name reference when adding the repository to a Git-based skills index or testing discovery locally.
