# Cheso Skills

Official Agent Skills for [Cheso](https://cheso.ai).

## Install

```bash
npx skills add cheso-ai-platform/skills --skill cheso -g -y
```

Omit `-g` for a project-level install, or pass `--agent <agent>` to target a specific agent.

## Setup (for AI agents)

When a user asks you to install or set up Cheso from this repository:

1. Install this skill globally (non-interactive):
   ```bash
   npx skills add cheso-ai-platform/skills --skill cheso -g -y
   ```
2. Read `skills/cheso/SKILL.md` and complete all steps in `## Setup and login`:
   - Install `cheso-cli` (`uv tool install cheso-cli -U`, or `pipx install cheso-cli`)
   - Check `cheso whoami --json` and authenticate if needed (`cheso login` with a browser, or `cheso login --device` in SSH/container; pass `--site` / `--base-url` only if specified so `CHESO_BASE_URL` or a remembered site is preserved)
   - Verify with `cheso whoami`, then let the user know when it is ready.
