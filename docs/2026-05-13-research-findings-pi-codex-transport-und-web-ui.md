# 2026-05-13 Research Findings — Pi Codex Transport und Web-UI

## 1. Meta

### 1.1 Zweck
Dieses Dokument hält die **Research Findings** fest, die während der Analyse von:
1. Pi-Codex-Transportverhalten
2. `packages/web-ui`-Check-/Typecheck-Problemen

ermittelt wurden.

### 1.2 Fokus
Es geht in diesem Dokument nicht primär um die Session-Chronologie, sondern um:
- lokale Code-/Config-Befunde
- Upstream-GitHub-/Repo-Befunde
- daraus abgeleitete technische Schlussfolgerungen

## 2. Research Scope

### 2.1 Pi-Runtime-Scope
Analysierte lokale Runtime-/Installationspfade:
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

### 2.2 Source-Fork-Scope
Analysierte Source-Dateien im Fork:
- `/home/jan/gh/pi-mono/packages/coding-agent/src/core/settings-manager.ts`
- `/home/jan/gh/pi-mono/packages/coding-agent/src/core/sdk.ts`
- `/home/jan/gh/pi-mono/packages/coding-agent/src/modes/interactive/interactive-mode.ts`
- `/home/jan/gh/pi-mono/packages/coding-agent/src/modes/interactive/components/settings-selector.ts`
- `/home/jan/gh/pi-mono/packages/agent/src/agent.ts`
- `/home/jan/gh/pi-mono/packages/agent/src/types.ts`
- `/home/jan/gh/pi-mono/packages/ai/src/types.ts`
- `/home/jan/gh/pi-mono/packages/ai/src/providers/openai-codex-responses.ts`
- `/home/jan/gh/pi-mono/packages/web-ui/example/tsconfig.json`
- `/home/jan/gh/pi-mono/packages/web-ui/package.json`
- `/home/jan/gh/pi-mono/packages/web-ui/tsconfig.json`
- `/home/jan/gh/pi-mono/packages/web-ui/tsconfig.build.json`
- `/home/jan/gh/pi-mono/packages/web-ui/example/src/custom-messages.ts`
- `/home/jan/gh/pi-mono/packages/web-ui/example/src/main.ts`
- `/home/jan/gh/pi-mono/packages/web-ui/src/index.ts`
- `/home/jan/gh/pi-mono/packages/web-ui/src/components/Messages.ts`

### 2.3 Upstream-Codex-Research-Scope
Lokaler read-only Upstream-Checkout:
- `/tmp/openai-codex-research`

Relevante Dateien:
- `/tmp/openai-codex-research/codex-rs/core/src/client.rs`
- `/tmp/openai-codex-research/codex-rs/core/tests/suite/websocket_fallback.rs`
- `/tmp/openai-codex-research/codex-rs/model-provider-info/src/lib.rs`
- `/tmp/openai-codex-research/codex-rs/core/config.schema.json`

### 2.4 GitHub-Issue-/PR-Scope
#### Codex
- `openai/codex#19821`
- `openai/codex#22156`
- `openai/codex#11682`
- `openai/codex#13041`
- `openai/codex#13103`
- `openai/codex#14297`

#### Pi / Web-UI
- `earendil-works/pi#2493`
- `earendil-works/pi#2840`
- `earendil-works/pi#2841`
- `earendil-works/pi#4387`
- `earendil-works/pi#4388`

## 3. Pi Codex Transport Research Findings

### 3.1 Lokaler Pi-Default-Befund
Im lokal installierten Pi-Runtime-Code war der generische Transportdefault:
- Runtime: `transport ?? "auto"`
- Doku: noch `"sse"`

Das ist ein klarer **Doc/Runtime mismatch**.

### 3.2 Pi-Provider-Befund
Im Pi-Codex-Provider war der relevante Pfad:
- effektiver Transport wird geprüft
- wenn nicht `sse`, wird der WebSocket-Pfad verwendet
- bei WebSocket-Fehlern existiert bereits session-bezogene Sticky-Fallback-Logik
- same-turn fallback ist nur sauber vor Streamstart sicher
- nach `after_message_stream_start` wird der Fehler hochgereicht statt replayt

### 3.3 Wichtigste lokale Root-Cause-Einschätzung
Pi hatte nicht nur ein allgemeines Upstream-WebSocket-Risiko, sondern auch ein **eigenes UX-/Control-Problem**:
- kein sauber eingebauter Codex-spezifischer Opt-out
- inkonsistente Default-Dokumentation
- kein klarer built-in Äquivalentpfad zu `supports_websockets = false`

## 4. Codex-CLI-Upstream Findings

### 4.1 WebSocket-Posture
Codex CLI ist **WS-first / WS-capable by default**.

Das ist kein Bauchgefühl, sondern wird strukturell gestützt durch:
- `supports_websockets: true` im Provider-Info-Modell
- `responses_websocket_enabled()` als Capability-Gate

### 4.2 Sticky Fallback
Codex CLI implementiert session-scoped sticky fallback:
- `disable_websockets`
- `force_http_fallback()`

Damit werden nach einem relevanten WS-Fehler weitere Turns derselben Session auf HTTP/SSE gehalten.

### 4.3 Special Case 426
`426 Upgrade Required` wird upstream explizit und direkt behandelt:
- immediate `FallbackToHttp`
- keine weitere unnötige Eskalation an dieser Stelle

### 4.4 Testgestützte Befunde
`websocket_fallback.rs` zeigt, dass upstream aktiv absichert:
- fallback bei `426`
- fallback nach Retry-Erschöpfung
- sticky fallback über Folgeturns

### 4.5 Grenzen des Upstream als Vorlage
Codex CLI ist eine gute **Transportstruktur-Vorlage**, aber nicht perfekt:
- mehrere offene WS-Issues sind noch aktiv
- insbesondere Connect-/Retry-/Disconnect-Fälle sind nicht vollständig „weggefixt“

Das heißt:
- als Policy-/Architecture-Vorlage: gut
- als Beweis, dass WS immer problemlos ist: nein

## 5. Wichtigste Architekturfolgerung aus dem Codex-Research
Pi sollte sich an Codex CLI angleichen in:
- WS-capable default posture
- capability gating
- sticky fallback
- klarer built-in websocket disable semantics

Pi sollte **nicht blind kopieren** in:
- same-turn replay nach bereits begonnenem Stream

Grund:
- Pi ist agentisch und tool-/side-effect-fähig
- Replay nach partieller Ausführung ist riskanter als bei einem rein chatförmigen Client

## 6. Web-UI Research Findings

### 6.1 Lokaler Fehlerbefund
`packages/web-ui/example/tsconfig.json` verwendete `dist/*.d.ts`-Pfade für lokale Typechecks.

Das macht Checks abhängig von bereits gebauten Artefakten und ist für Clean-Checkout-Validierung fragil.

### 6.2 Export-Surface-Befund
Die initialen `TS2307`-Fehler im Example bedeuteten **nicht automatisch fehlende Exporte**. `packages/web-ui/src/index.ts` exportiert die relevanten Symbole.

Das Problem lag primär in der **TypeScript-Resolution-/Build-Order-Konfiguration**, nicht in einem leeren Export-Surface.

### 6.3 Historischer Upstream-Befund
Upstream hatte genau dieses Problem bereits erkannt und adressiert:
- `#2493`: clean-workspace checks gegen `dist`-Abhängigkeit härten
- `#2840`/`#2841`: web-ui checks wiederherstellen, source-basierte Auflösung und `Messages.ts`-Anpassung

### 6.4 Regression-Hypothese
Ein späterer Commit stellte Example-Pfade erneut auf `dist`-basierte Auflösung. Das ist starke Evidenz für eine **Regression**, nicht nur ein lokales Missverständnis.

### 6.5 Wiederhergestellte Korrekturrichtung
Die korrekte Richtung für den Example-Typecheck ist source-basiert:
- `../../agent/src/index.ts`
- `../../ai/src/index.ts`
- `../../tui/src/index.ts`
- `../src/index.ts`

## 7. Maintainer-/Repo-Wechsel Findings

### 7.1 Kanonisches Repo
`badlogic/pi-mono` redirectet auf `earendil-works/pi`.

### 7.2 Bedeutet das für die Analyse?
Ja, es erklärt alte Referenzen im Paket-/Repo-Metadatenbestand.

Aber:
- der aktuelle Web-UI-Fehler ist im **neuen/aktuellen Repo** selbst wieder beobachtbar gewesen
- also nicht bloß ein Artefakt eines falschen Review-Ziels

## 8. Research-basierte Schlussfolgerungen

### 8.1 Für Pi-Codex-Transport
- Pi brauchte ein **Fix/Update**, kein alternatives Produktverhalten.
- Der richtige Fix ist **WS-first + built-in disable switch + sticky fallback + no-replay-after-stream-start**.

### 8.2 Für `packages/web-ui`
- Das Checkproblem war bekannt, historisch fixbar und wurde plausibel re-regressiert.
- Source-basierte Auflösung ist der richtige lokale Typecheckpfad.

## 9. Offene Research-Grenzen
- [ ] Kein vollständiger line-by-line Review jeder einzelnen Low-Level-WebSocket-Datei von Codex CLI wurde durchgeführt.
- [ ] Kein frischer live reproduzierter WS-Fehler im finalen WS-first-Pi-Fixlauf.
- [ ] Kein zusätzlicher externer Maintainer-Kommentar jenseits der verfügbaren GitHub-Issues/PRs eingeholt.

## 10. Reuse-Empfehlung
Diese Research Findings sollten wiederverwendet werden für:
- Rebase/Neuportierung gegen spätere Maintainer-Refactors
- spätere fokussierte Tests für `codexWebsocketsEnabled`
- Web-UI-Clean-Checkout-Regressionstests

## 11. Actionable Next Steps
- [ ] Focused regression tests für `codexWebsocketsEnabled` ergänzen.
- [ ] Sticky fallback live erneut provozieren oder testgestützt simulieren.
- [ ] Bei Maintainer-Updates am 16./17. die drei Session-Commits gegen den neuen Stand neu validieren.
