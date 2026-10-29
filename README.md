# Flutter Offline AI Master Skill

A project-level coding-agent skill and reference pack for implementing **real, testable on-device AI in existing Flutter apps**. It uses Flutter Edge AI as the reference architecture and focuses on avoiding common integration failures rather than only generating sample code.

**Research snapshot:** 29 October 2026. APIs, packages, supported model artifacts and device requirements change; use the linked official docs and the versions resolved by the target app before coding.

## What is included

- `SKILL.md` — master instructions for an AI coding agent.
- `references/` — repository audit, package architecture, engine/model selection, implementation flow, offline acceptance testing, troubleshooting, lifecycle/security and API traps.
- `prompts/INTEGRATION_TASK_PROMPT.md` — a copy/paste task prompt to run with the skill in an existing repository.

## Install into Codex / Antigravity

1. Extract this ZIP.
2. Copy the whole extracted folder (or rename it `flutter-offline-ai-master`) into your project at:

   `.agents/skills/flutter-offline-ai-master/`

   Result should be `.agents/skills/flutter-offline-ai-master/SKILL.md` plus its `references/` and `prompts/` folders. Codex and Antigravity use `.agents/skills/` in the documented convention.
3. Restart or refresh the coding agent if it does not discover new skills.
4. Open your existing Flutter repository and ask it to inspect and implement offline AI using the master skill.
5. Follow the integration prompt in `prompts/INTEGRATION_TASK_PROMPT.md`.

Other assistant locations from Flutter Edge AI's official package-skills guide:

- Claude Code: `.claude/skills/flutter-offline-ai-master/`
- Cursor: `.cursor/skills/flutter-offline-ai-master/`
- GitHub Copilot: `.github/skills/flutter-offline-ai-master/`
- Cline: `.cline/skills/flutter-offline-ai-master/`
- OpenCode: `.opencode/skills/flutter-offline-ai-master/`

Keep the folder name and `SKILL.md` together; reference links inside the skill are relative to it.

## Install the source package's version-matched skills too

In the Flutter app root, first add the core and only the selected engine. For LiteRT-LM, for example:

```bash
flutter pub add flutter_edge_ai flutter_edge_ai_litertlm
```

Then install the package's own up-to-date coding-agent skills:

```bash
dart run skills@ get --all --agent codex
```

Use `--agent claude`, `cursor`, `copilot`, `antigravity`, etc. as appropriate. If you have not yet selected an agent, see the official guide. These installed skills are useful because they track the installed package version; this ZIP is a cross-project master workflow and may need refresh when APIs change.

## Use safely

- This ZIP does not modify your app or download a model.
- Do not add every runtime by default. Select one engine that supports the exact target model and platform.
- Treat the model file size, runtime RAM, and quality as different things.
- Require offline tests after initial model setup. A local model alone does not make remote tools or network APIs offline.
- Do not hard-code a Hugging Face token or any secret into a mobile app.

## Official sources

See `references/09-official-sources.md` for the canonical source index.
