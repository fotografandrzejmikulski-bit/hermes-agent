# OpenAI Skills Architecture for Hermes Android

## Status

This is the Android integration contract for OpenAI-compatible Skills. It deliberately separates procedural skill instructions from execution tools and from the Android UI/runtime.

## Runtime topology

```text
Android UI
   |
   v
Android agent facade
   |
   +--> local device capability adapters
   |
   +--> remote Hermes API / Responses-compatible provider
   |
   +--> on-device model backends
   |
   v
Hermes Agent runtime
   |
   +--> skill selection / loading
   |      |
   |      +--> skills/<name>/SKILL.md
   |      +--> references/
   |      +--> assets/
   |      +--> scripts/
   |
   +--> tool router
          |
          +--> built-in tools
          +--> configured MCP tools
```

## Responsibilities

### Android

- Present chat and agent state.
- Manage Android lifecycle and foreground execution where required.
- Request and enforce OS permissions.
- Expose narrow device capabilities through audited adapters.
- Store only required local state and credentials using Android-secure storage.
- Never embed provider API keys in the APK.

### Hermes runtime

- Select and load skills.
- Perform planning and tool orchestration.
- Maintain conversation/session state.
- Enforce tool and command policies.
- Execute configured MCP integrations where supported by the runtime.
- Return verifiable results to the Android client.

### MCP

MCP is the controlled execution boundary for external capabilities. A skill may describe how an MCP tool should be selected and used, but the skill does not replace the MCP server or manufacture its results.

The current embedded Android path documents external MCP stdio/SSE/HTTP execution as unavailable. Do not enable it merely by adding a skill; it requires a lifecycle-safe Android MCP transport implementation and explicit validation.

## Skill contract

Every new production skill should contain:

```text
skills/<name>/
├── SKILL.md
├── DESCRIPTION.md          # when Hermes skill discovery requires it
├── references/              # optional
├── assets/                  # optional
└── scripts/                 # optional, deterministic only
```

`SKILL.md` must define trigger conditions, input assumptions, workflow, tool boundaries, security constraints, validation, failure behavior, output expectations, and stop conditions.

## OpenAI plugin packaging boundary

The OpenAI plugin `agents/openai.yaml` dependency declaration is a deployment/submission artifact, not an Android runtime configuration file. It must reference a real MCP deployment and authoritative URL. Placeholder MCP URLs are prohibited.

When a real MCP server exists, package it separately and declare the dependency there; keep the Android application independent of plugin submission metadata.

## Validation gates

A feature is not considered integrated merely because files exist. The minimum evidence for a production change is:

1. Skill discovery succeeds.
2. The intended workflow activates for a representative direct request.
3. An indirect equivalent request activates the same workflow when appropriate.
4. Incomplete input follows the defined follow-up path.
5. Unsupported operations do not activate the skill or do not execute an unavailable tool.
6. Android build/lint/tests pass for affected code.
7. Device lifecycle behavior is validated for long-running work.
8. Secrets are absent from source, logs, packaged resources, and test fixtures.

## Current limitation

The existing Android application already supports embedded local and remote model paths, but its documented embedded runtime does not execute external MCP stdio/SSE/HTTP sessions. The first implementation phase therefore establishes the portable Skill contract and Android/runtime boundary without falsely claiming live MCP execution.
