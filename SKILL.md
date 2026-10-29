---
name: flutter-offline-ai-master
description: Master workflow for auditing, integrating, debugging, and production-validating real on-device/offline AI in existing Flutter apps, using Flutter Edge AI as a reference implementation. Use when offline LLM, local inference, model download/loading, streaming, tools, RAG, speech, or device-only AI in Flutter is failing or must be added.
---

# Flutter Offline AI Master Skill

## Mission

Make on-device AI actually work in the user's existing Flutter application—not merely compile, display a demo response, or appear to work while the network is available. Use the official Flutter Edge AI project as a primary reference architecture, but choose APIs and packages only after checking the project's current SDK constraints and the current official documentation.

The expected outcome is a small, maintainable integration that is tested on a real target device with networking disabled after model setup. Protect existing app behavior, architecture, stored data, authentication, business logic, and UI unless the task explicitly requires changes.

## When to use this skill

Use for requests involving:
- Running an LLM locally in Flutter Android/iOS apps.
- Making an AI assistant/chat work without internet after model installation.
- Integrating Flutter Edge AI, LiteRT-LM, llama.cpp/GGUF, ONNX Runtime, MediaPipe, or an OS-provided on-device model.
- Diagnosing model download, loading, inference, empty response, slow response, OOM, accelerator fallback, app lifecycle, or offline failures.
- Adding streaming chat, system instructions, function calling, local embeddings/RAG, speech, or on-device agent skills.
- Auditing an existing implementation that is reported to work only on Wi-Fi, only on an emulator, or only on the first screen.

## Source-of-truth policy

1. Inspect the actual source code and project configuration before proposing a design. Do not infer implementation from Markdown docs, filenames, README claims, screenshots, or the user's description alone.
2. Check current official docs, package README/changelog, and the exact version resolved by `pubspec.lock`. Flutter Edge AI changes over time; never treat examples in this skill as more authoritative than current docs and installed APIs.
3. Read the package-bundled coding-agent skills when available. With `flutter_edge_ai` 2.0.0+ in the app, run `dart run skills@ get --all` (or add an explicit `--agent <name>` if detection fails), then inspect the installed `flutter-edge-ai-inference` skill and relevant feature skills. This master skill supplements, not replaces, those version-matched references.
4. Prefer current package APIs over old `flutter_gemma*` APIs. Flutter Gemma was renamed to Flutter Edge AI; do not mix imports, dependency names, or examples from different generations.
5. Never invent benchmark numbers, supported models, minimum SDK versions, model URLs, backend support, or test results. Record what was verified and on which target.

## Non-negotiable engineering rules

- **Audit before editing.** Map the Flutter/Dart version, platform targets, state-management approach, architecture, current AI packages, model source/format, service boundaries, tests, and existing failures.
- **Minimum surface area.** Add only the core package and the one engine required for the selected model. Add speech/RAG/agent/diagnostics packages only when their features are in scope.
- **One engine first.** Prove model install → model load → one prompt → streamed or complete response → app restart → offline prompt. Only then add more features.
- **Do not silently switch the user's model or architecture.** If the selected model is incompatible, explain the evidence and propose the smallest compatible change. Preserve product requirements and existing state management.
- **Do not call it offline merely because inference is local.** Verify the full user flow, including stored model files, startup, prompts, conversation persistence if required, and every invoked tool, with connectivity disabled.
- **Do not lie about success.** Separate static analysis, unit tests, device integration tests, and manual offline tests. If no physical device was available, explicitly state that hardware verification remains unproven.
- **Secrets stay out of source code.** Use environment/build-time configuration or a safe secret store; do not commit Hugging Face tokens or other credentials. Never bundle a private token in a public mobile binary.
- **No dangerous tool execution.** Validate model-generated arguments, enforce allowlists and permission checks, confirm destructive/sensitive actions, and return errors as tool results rather than crashing the chat loop.
- **Clean lifecycle.** Prevent duplicate model initialization, prevent concurrent generations unless deliberately supported, close sessions/models at the correct ownership boundary, handle widget disposal, cancellation, and background/foreground transitions.
- **No unrelated cleanup.** Do not refactor unrelated screens, remove files, or upgrade unrelated dependencies as part of AI integration.

## Required workflow

### Phase 1 — Establish the baseline

Inspect at least:
- `pubspec.yaml`, `pubspec.lock`, `analysis_options.yaml`, Flutter/Dart SDK constraints.
- `lib/` service/repository/state-management layer and all AI call sites.
- Android Gradle/minSdk/Kotlin config and iOS deployment target/Podfile/SPM setup if those platforms are targeted.
- Existing model assets, URL/configuration, download/storage logic, permissions and platform-specific files.
- Test folders, CI workflows, and the current app startup/lifecycle.

Run baseline checks appropriate to the repository before edits (`flutter --version`, `dart --version`, `flutter pub get`, `flutter analyze`, relevant tests). Preserve existing failures in a baseline note so they are not incorrectly attributed to the change.

Produce a short audit with: observed current flow, confirmed root causes, compatibility constraints, files likely to change, risks, tests to run, and explicitly out-of-scope work. If implementation was requested, continue after the audit without asking unnecessary questions. Ask a focused question only when a decision genuinely blocks a safe implementation (for example, mutually incompatible product requirements or a destructive migration).

### Phase 2 — Define what “offline” means for this app

Write testable acceptance criteria, selecting only those relevant:
- First-run model installation needs network OR model is intentionally bundled/preloaded.
- Installation progress/error/retry and storage checks are visible.
- Model file remains present after app restart and is not downloaded again on every launch.
- Model initializes when connectivity is absent.
- A user prompt produces a non-empty response with connectivity disabled.
- No inference request, prompt, or private conversation is sent to a cloud endpoint.
- Any remote-only feature is disabled, queued, or clearly marked unavailable offline.
- If chat history must persist, it is stored locally with explicit privacy/retention rules.
- Permission-denied, corrupt/missing model, unsupported device, low memory, interrupted download, and app-background cases have sensible recovery.

Differentiate **offline inference**, **offline first launch**, **offline chat history**, and **offline tools**. A network-down test after a successful model download proves the first, not automatically the other three.

### Phase 3 — Choose the model/runtime from evidence

1. Identify exact model repo, filename/format, architecture, license, download size, quantization, tokenizer/chat template, supported modalities, tool-calling support, and target device memory.
2. Match model format to engine explicitly:
   - `.litertlm` → `flutter_edge_ai_litertlm` / `LiteRtLmEngine`.
   - `.gguf` → `flutter_edge_ai_llama` / llama.cpp.
   - `.task` or `.bin` → `flutter_edge_ai_mediapipe` / `MediaPipeEngine` (maintenance-only for the Google LLM Inference API; prefer `.litertlm` for new apps when a compatible build exists).
   - ONNX → `flutter_edge_ai_onnx`; distinguish native ORT-GenAI model directory from browser Transformers.js path.
   - OS-provided model → `flutter_edge_ai_builtin_ai`; device/OS-gated and not equivalent to shipping your own arbitrary model.
3. Validate model family → `ModelType`, and always declare the exact `ModelFileType`. Never rely on filename extension inference. `installModel` defaults to `.task`; omitting `fileType: ModelFileType.litertlm` can route a `.litertlm` model to the wrong engine.
4. Check the current official Models and Capabilities tables. A model may support image/audio/function calling in one exported artifact but not another. The existence of a model page does not prove it works on every platform/backend.
5. Choose the smallest model that meets the quality requirement and has enough headroom on actual target devices. Do not infer runtime RAM from download size. Benchmark a real mid/low-range phone plus a higher-end phone when possible.
6. If model selection is uncertain, build a tiny benchmark screen or isolated spike before integrating into core app flows.

### Phase 4 — Integrate into the existing app

- Keep AI responsibilities inside a dedicated service/repository (or the app's established equivalent), not inside multiple widgets. Keep presentation state in the project's existing state-management pattern.
- Separate responsibilities: runtime initialization, model acquisition/install, model readiness, model/session lifecycle, prompt/response generation, persistence, and optional feature-specific tools.
- Use a guarded one-time initialization path. Avoid initializing/loading a model on every widget build or navigation event.
- Model download is a first-run/setup operation; normal app startup should detect the installed model and open it instead of redownloading it.
- Add progress and errors as real state transitions, not indefinite spinners. Provide retry and cancellation where supported, and protect against stale async results after widget disposal.
- Keep the first implementation text-only unless vision/audio is specifically required. Additional modalities affect memory, model bundle, API flags, model template and test coverage.
- Ensure inference does not block the UI. Use the package's async/streaming APIs, surface partial output promptly, and support stop/cancel if appropriate.
- Persist chat history only if product requirements need it; do not confuse the model runtime's session memory with durable app storage.
- Check actual active backend when CPU/GPU/NPU matters. If acceleration is required, do not silently accept CPU fallback as a successful performance result.
- Make model ID/source, engine, model type, file type, context budget and output cap explicit configuration values. Avoid hard-coded credentials.

### Phase 5 — Verify feature-by-feature

Test the narrow happy path before every optional feature:
1. Model installs completely; progress reaches completion; a retry after interruption behaves correctly.
2. Model is discovered after process death/restart.
3. A simple non-empty text response works on the target device.
4. Airplane mode/network disabled: restart the app and generate another response.
5. Long prompt, longer conversation, repeated questions and Stop/cancel behavior.
6. Small-memory/low-end device behavior and model load failures.
7. If tools are needed: declare one harmless tool, validate arguments, execute it, feed its result back, handle malformed and error responses.
8. If RAG is needed: persist index, restart app, search relevant content, verify embedding profile and storage remain consistent.
9. If speech is needed: verify sampling format (STT expects raw 16 kHz mono PCM on the documented path; WAV/48 kHz input may yield wrong transcripts without being rejected), permissions, latency, and offline operation.
10. Test Android and iOS separately; one platform passing does not validate the other. Use a physical iOS device for GPU/memory behavior when Simulator limits affect the model.

### Phase 6 — Run checks and report truthfully

Run the checks available in the user's project:
- `flutter pub get`
- `dart format` on changed Dart files
- `flutter analyze`
- focused unit/widget tests
- integration tests on supported target devices
- release/profile build where practical
- a manual network-disabled acceptance test after initial model setup

Review the final diff. Confirm no secrets, accidentally bundled multi-GB model, unrelated changes, missing platform config, unbounded retries, or lifecycle regressions. Summarize changed files, root cause fixed, package/engine/model selected, commands and results, manual-device evidence, known limitations, and remaining unverified items. Never report tests as passing if they were not run.

## Known high-impact Flutter Edge AI traps

1. `Message.isUser` defaults to `false`. Always set `isUser: true` for user messages; otherwise a prompt may produce an empty response.
2. `maxTokens` is the complete context window (system prompt + conversation + generated tokens), not reply length. Use `maxOutputTokens` for the response cap where the selected engine supports it.
3. `.litertlm` and `.task` are different formats routed to different engines. Specify `fileType` explicitly.
4. Engine packages do not re-export core imports. Import `package:flutter_edge_ai/flutter_edge_ai.dart` as well as the engine package.
5. `flutter_edge_ai` core does not register an engine by itself. Register the selected engine in `FlutterEdgeAi.initialize(...)`.
6. Current 2.x APIs differ from `flutter_gemma*` tutorials. Check current migration docs; don't paste old names or removed APIs.
7. RAG moved out of core in 2.x into `flutter_edge_ai_rag`, with vector-store providers such as SQLite or Qdrant. Follow current APIs, not old `FlutterEdgeAi.rag` examples.
8. Inference may be local while a skill/tool still requires the internet. Audit every network call in the complete flow.
9. Multiple independent sessions still serialize generation in the documented core path and add context memory. Avoid many sessions on lower-memory phones.
10. A requested GPU can silently fall back to CPU on some configurations. Inspect `activeBackend` and logs; use the engine's documented no-fallback option where diagnosing supported engines.
11. On `.litertlm` native paths, context sizes below 1024 are clamped or unsuitable; consult current engine documentation. Reduce output with `maxOutputTokens`, not by shrinking the context budget to an invalid value.
12. Web has different export, storage, API and memory constraints than Android/iOS. Never use a native model artifact on Web just because the same extension is accepted by the installer.

## Delegation and reference reading

Read only the relevant supporting reference(s), then the authoritative current documentation for the selected runtime:
- `references/01-project-audit.md`
- `references/02-architecture-and-packages.md`
- `references/03-model-and-runtime-selection.md`
- `references/04-implementation-playbook.md`
- `references/05-offline-acceptance-tests.md`
- `references/06-troubleshooting.md`
- `references/07-production-security-and-lifecycle.md`
- `references/08-verified-api-patterns.md`
- `references/09-official-sources.md`

## Definition of done

The task is not complete until all applicable statements are true:
- [ ] Existing project and SDK constraints were inspected before editing.
- [ ] Exact engine, model artifact, `ModelType`, and `ModelFileType` match.
- [ ] Only required dependencies and platform changes were added.
- [ ] Model installation, persistence, initialization, response, and cleanup have explicit states.
- [ ] User prompts produce non-empty responses; streaming/error/cancel behavior is handled.
- [ ] Model successfully answers after app restart with connectivity disabled (if a target device was available).
- [ ] Relevant tests/build/analyzer checks have actual results.
- [ ] Security, privacy, tool permissions, memory, and download failure modes were reviewed.
- [ ] Final report distinguishes verified results from untested assumptions.
