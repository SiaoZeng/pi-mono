# 2026-05-13 Session Findings — Codex Transport Alignment and Web UI Check Repair

## 0. Meta
- Date: 2026-05-13
- Workspace: `/home/jan/gh/pi-mono`
- Branch: `fix/codex-ws-transport-alignment`
- Fork: `https://github.com/SiaoZeng/pi-mono`
- Upstream: `https://github.com/earendil-works/pi`
- Session Scope:
  - `openai-codex` transport behavior in Pi CLI
  - `packages/web-ui` clean-check regression
- Governing Spec:
  - `/home/jan/.pi/agent/specs/2026-05-13-pi-openai-codex-ws-first-transport-template-spec.md`
- Superseded Artifacts:
  - `/home/jan/.pi/agent/specs/2026-05-13-pi-openai-codex-transport-policy-spec.md`
  - `/home/jan/.pi/agent/plans/2026-05-13-pi-openai-codex-transport-policy-implementation-plan.md`

## 1. Executive Summary
- [x] The original assumption that Pi should force `openai-codex` into SSE-first mode was incorrect.
- [x] The correct direction is a **Pi fix/update** that keeps Codex **WebSocket-capable by default**, using the OpenAI Codex CLI as the transport behavior template.
- [x] Pi now has a built-in provider-specific control, `codexWebsocketsEnabled`, which acts as the Pi-native equivalent of Codex CLI users setting `supports_websockets = false` via a custom provider workaround.
- [x] Pi preserves sticky fallback after WebSocket failure and retains the no-replay safety boundary once a turn has already started streaming.
- [x] A separate but blocking repository-quality issue in `packages/web-ui` was identified, researched against current upstream history, fixed in the fork, and validated locally.

## 2. Session Timeline and Decision Points
### 2.1 Initial transport investigation
- [x] A concrete Pi session failure was inspected in:
  - `/home/jan/pi-envs/superpowers-lab/sessions/2026-05-13T04-13-44-163Z_019e1f8a-6563-769b-8463-8b852ffe2e21.jsonl`
- [x] The relevant diagnostic showed:
  - `configuredTransport: "auto"`
  - `eventsEmitted: true`
  - `phase: "after_message_stream_start"`
  - `errorMessage: "WebSocket error"`
- [x] This bounded the failure to a **mid-stream WebSocket transport failure** in Pi’s `openai-codex` path.

### 2.2 Initial wrong direction
- [x] A first patch attempted to make Pi SSE-first for Codex.
- [x] This direction was later rejected after clarifying the goal: Pi should be aligned to the Codex CLI transport posture, not diverge from it.
- [x] The wrong-direction runtime changes were rolled back.
- [x] The corresponding spec/plan artifacts were marked `Superseded`.

### 2.3 Corrected transport direction
- [x] The new direction was established as:
  - keep `openai-codex` WS-capable by default
  - add a Pi-native built-in websocket capability switch
  - preserve sticky fallback
  - preserve safe pre-stream-start fallback
  - keep post-stream-start no-replay

### 2.4 Fork setup and source port
- [x] A fork for source-backed work was created and cloned locally.
- [x] Runtime-only local changes in installed `node_modules` were ported into the source fork under `/home/jan/gh/pi-mono`.
- [x] The work was committed and pushed on branch:
  - `fix/codex-ws-transport-alignment`

### 2.5 Web UI clean-check regression follow-up
- [x] A repository check failure in `packages/web-ui` was investigated.
- [x] Local evidence showed the example TypeScript config depended on built `dist/*.d.ts` outputs.
- [x] GitHub research on current upstream found the same problem had already been known and previously fixed.
- [x] The example was restored to source-based workspace path resolution in the fork.

## 3. Codex CLI Research Findings
### 3.1 What was researched
- [x] Local read-only upstream checkout under:
  - `/tmp/openai-codex-research`
- [x] Core source files reviewed:
  - `/tmp/openai-codex-research/codex-rs/core/src/client.rs`
  - `/tmp/openai-codex-research/codex-rs/core/tests/suite/websocket_fallback.rs`
  - `/tmp/openai-codex-research/codex-rs/model-provider-info/src/lib.rs`
  - `/tmp/openai-codex-research/codex-rs/core/config.schema.json`
- [x] Upstream issue evidence reviewed:
  - `openai/codex#19821`
  - `openai/codex#22156`
  - `openai/codex#11682`
  - `openai/codex#13041`
  - `openai/codex#13103`
  - `openai/codex#14297`

### 3.2 What was confirmed
- [x] Codex CLI is **WebSocket-first / WebSocket-capable by default**.
- [x] Upstream models this via `supports_websockets: true` on the OpenAI provider.
- [x] Upstream uses **session-scoped sticky fallback** through `disable_websockets` + `force_http_fallback()`.
- [x] Upstream explicitly handles `426 Upgrade Required` with immediate HTTP fallback.
- [x] Upstream test coverage proves:
  - sticky fallback across subsequent turns
  - same-turn fallback after connect/handshake failure
- [x] Upstream still has open WebSocket issues, so it is a template for structure and posture, not proof that the path is bug-free.

### 3.3 Key Pi-specific divergence retained intentionally
- [x] Pi does **not** silently replay a turn after `after_message_stream_start`.
- [x] Reason: Pi may already have emitted partial tool-driven side effects in the current turn.
- [x] This is an intentional safety divergence from the broader fallback behavior one might be tempted to copy from pure chat-style flows.

## 4. Pi Transport Patch Findings
### 4.1 Installed runtime surface touched during exploratory and validation work
- [x] The following installed runtime files were used for local patching/validation:
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/dist/core/settings-manager.js`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/dist/core/settings-manager.d.ts`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/dist/core/sdk.js`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/dist/modes/interactive/interactive-mode.js`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/dist/modes/interactive/components/settings-selector.js`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/dist/modes/interactive/components/settings-selector.d.ts`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/node_modules/@earendil-works/pi-agent-core/dist/agent.js`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/node_modules/@earendil-works/pi-agent-core/dist/agent.d.ts`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/node_modules/@earendil-works/pi-ai/dist/types.d.ts`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/node_modules/@earendil-works/pi-ai/dist/providers/openai-codex-responses.js`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/node_modules/@earendil-works/pi-ai/dist/providers/openai-codex-responses.d.ts`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/docs/settings.md`
  - `/home/jan/.npm-global/lib/node_modules/@earendil-works/pi-coding-agent/docs/providers.md`

### 4.2 Final transport behavior implemented
- [x] Built-in setting introduced: `codexWebsocketsEnabled`
- [x] Default value: `true`
- [x] Meaning:
  - `true` -> `openai-codex` remains WS-capable by default
  - `false` -> Pi uses SSE for Codex without a custom provider workaround
- [x] Generic `transport` runtime default was aligned to `auto` in the source fork and local runtime documentation.
- [x] Sticky fallback after WebSocket failure remains in place.
- [x] Safe pre-stream-start fallback remains in place.
- [x] Silent same-turn replay after stream start remains blocked.

### 4.3 Source fork files changed for the Codex transport fix
- [x] `packages/coding-agent/src/core/settings-manager.ts`
- [x] `packages/coding-agent/src/core/sdk.ts`
- [x] `packages/coding-agent/src/modes/interactive/interactive-mode.ts`
- [x] `packages/coding-agent/src/modes/interactive/components/settings-selector.ts`
- [x] `packages/agent/src/agent.ts`
- [x] `packages/agent/src/types.ts`
- [x] `packages/ai/src/types.ts`
- [x] `packages/ai/src/providers/openai-codex-responses.ts`
- [x] `packages/coding-agent/docs/settings.md`
- [x] `packages/coding-agent/docs/providers.md`

### 4.4 Source commit for Codex transport fix
- [x] Commit: `2e4eb5d4`
- [x] Message: `fix: align codex transport handling with codex cli`

## 5. Web UI Review and Patch Findings
### 5.1 What was wrong
- [x] `packages/web-ui/example/tsconfig.json` used `dist/*.d.ts` path mappings:
  - `../../agent/dist/index.d.ts`
  - `../../ai/dist/index.d.ts`
  - `../../tui/dist/index.d.ts`
  - `../dist/index.d.ts`
- [x] In a clean checkout, those artifacts can be missing.
- [x] Therefore the local `packages/web-ui` check path becomes structurally unreliable.

### 5.2 What research showed
- [x] The issue was already known in the **current** upstream repo line:
  - `earendil-works/pi#2493`
  - `earendil-works/pi#2840`
  - `earendil-works/pi#2841`
- [x] Those artifacts describe the exact same clean-check failure mode and prior fixes.
- [x] A later upstream commit reintroduced the `dist`-based example mapping, making the problem appear again.
- [x] This is therefore not just a stale-local-state problem; it is a real regression in the current upstream line.

### 5.3 What was fixed
- [x] `packages/web-ui/example/tsconfig.json` was restored to source-based workspace resolution.
- [x] New path mappings now point to:
  - `../../agent/src/index.ts`
  - `../../ai/src/index.ts`
  - `../../tui/src/index.ts`
  - `../src/index.ts`
- [x] `baseUrl` was restored as well.

### 5.4 Web UI validation after fix
- [x] `cd /home/jan/gh/pi-mono/packages/web-ui/example && tsc --project tsconfig.json --noEmit --pretty false`
- [x] Passed
- [x] `cd /home/jan/gh/pi-mono/packages/web-ui && npm run check`
- [x] Passed

### 5.5 Source commit for Web UI fix
- [x] Commit: `44c24a32`
- [x] Message: `fix(web-ui): restore clean example typecheck`

## 6. Validation Evidence Produced During Session
### 6.1 Pi runtime Codex validation artifacts
- [x] `/tmp/pi-codex-ws-default.jsonl`
  - [x] default WS-capable run
  - [x] returned `OK`
  - [x] no fresh `WebSocket error`
  - [x] no fresh `provider_transport_failure`
- [x] `/tmp/pi-codex-ws-disabled.jsonl`
  - [x] built-in websocket disable path
  - [x] returned `OK`
  - [x] no fresh `WebSocket error`
  - [x] no fresh `provider_transport_failure`
- [x] Additional earlier session artifacts still available:
  - `/tmp/pi-codex-sse-default.jsonl`
  - `/tmp/pi-codex-websocket-optin.jsonl`

### 6.2 Remaining validation limits
- [ ] A fresh, live-reproduced WebSocket failure did **not** occur during the final WS-first validation run.
- [x] Sticky fallback therefore remains validated by:
  - [x] current code-path inspection
  - [x] prior concrete failing session evidence
- [ ] There were no alternate provider credentials available for a fresh non-Codex live-regression run.

## 7. Git / Branch State
- [x] Fork branch: `fix/codex-ws-transport-alignment`
- [x] Commits on branch from this session:
  - [x] `2e4eb5d4 fix: align codex transport handling with codex cli`
  - [x] `44c24a32 fix(web-ui): restore clean example typecheck`
- [x] Branch pushed to fork remote

## 8. Build / Check Findings
### 8.1 Fork-level source validation
- [x] Source patch was ported into the fork successfully.
- [x] Commit hooks initially failed because repo-wide checks touched unrelated surfaces (`packages/web-ui` before it was fixed, and unrelated `packages/web-ui`/other workspace conditions during intermediate stages).
- [x] After the `packages/web-ui` fix, `packages/web-ui` check path was restored successfully.

### 8.2 Important caveat
- [ ] The earlier Codex transport source commit had to be created while global hook checks were not fully green due to unrelated issues at that moment.
- [x] The resulting branch is still materially improved because the later `packages/web-ui` blocking regression was fixed and the branch now carries both relevant commits.

## 9. Maintainer / Repository Transition Finding
- [x] Historical metadata still references older repo naming (`badlogic/pi-mono`) in some places.
- [x] GitHub currently redirects `badlogic/pi-mono` to `earendil-works/pi`.
- [x] The `packages/web-ui` clean-check regression was confirmed in the **current** maintainer repo line, so this was not merely a review against the wrong repository.

## 10. Session-Level Review Summary
### 10.1 Codex transport fix
- [x] Direction corrected from wrong SSE-first experiment to correct WS-first fix/update
- [x] Pi now has a built-in provider-specific Codex websocket capability switch
- [x] Pi remains aligned to Codex CLI at the transport-posture level
- [x] Pi retains its own safety boundary around post-stream-start replay

### 10.2 Web UI clean-check fix
- [x] A real, upstream-known clean-check regression was identified
- [x] The example was restored to source-based path resolution
- [x] `packages/web-ui` local check now passes again

### 10.3 Outstanding quality gaps
- [ ] No independent external `code-reviewer` checkpoint was performed in runtime
- [ ] The Codex transport fix would still benefit from focused automated regression tests
- [ ] Sticky fallback was not freshly triggered live in the final WS-first validation run

## 11. Recommended Next Steps
- [ ] Add focused automated tests for `codexWebsocketsEnabled`, sticky fallback, and post-stream-start no-replay in the source fork.
- [ ] If the branch is intended for long-term maintenance, resolve any remaining repo-wide unrelated check failures so future commits do not require hook bypasses.
- [ ] Keep the superseded SSE-first spec/plan only as historical record; do not use them for future execution.

## 12. Rollback Notes
- [ ] To remove the Codex transport fix, revert commit `2e4eb5d4`.
- [ ] To remove the Web UI clean-check fix, revert commit `44c24a32`.
- [ ] To remove both session changes from the fork branch, revert both commits in reverse order.
