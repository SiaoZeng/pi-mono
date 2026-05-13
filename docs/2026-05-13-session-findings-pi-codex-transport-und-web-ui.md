# 2026-05-13 Session Findings — Pi Codex Transport und Web-UI

## 1. Meta

### 1.1 Kontext
- Projektordner: `/home/jan/gh/pi-mono`
- Fork-Owner: `SiaoZeng`
- Upstream: `https://github.com/earendil-works/pi`
- Arbeitsbranch: `fix/codex-ws-transport-alignment`
- Relevante Governing Spec:
  - `/home/jan/.pi/agent/specs/2026-05-13-pi-openai-codex-ws-first-transport-template-spec.md`

### 1.2 Session-Ziel
Diese Session hatte zwei fachlich zusammenhängende Ziele:
1. Pi-CLI im `openai-codex`-Transportpfad an das relevante Codex-CLI-Verhalten angleichen.
2. Den blockierenden `packages/web-ui`-Checkfehler analysieren und im Fork beheben.

### 1.3 Wichtigste Korrektur der Session
Der erste Lösungsansatz war fachlich falsch ausgerichtet: Pi wurde kurzfristig in Richtung `SSE-first` für Codex verändert. Nach erneuter Klärung des Ziels wurde dieser Ansatz vollständig verworfen und zurückgesetzt.

Die korrekte Zielrichtung ist:
- `openai-codex` bleibt in Pi standardmäßig **WebSocket-capable / WS-first-kompatibel**.
- Pi erhält eine **built-in Codex-spezifische WebSocket-Capability-Steuerung**.
- Pi übernimmt **sticky fallback** und relevante Upstream-Ideen aus Codex CLI.
- Pi behält bewusst die Pi-spezifische Sicherheitsgrenze: **kein silent replay nach `after_message_stream_start`**.

## 2. Session-Verlauf

### 2.1 Phase 1 — Lokale Fehlerbeobachtung und erste Eingrenzung
Ausgangspunkt war eine Pi-Session mit realem Codex-Transportfehler:
- Session-Artefakt:
  - `/home/jan/pi-envs/superpowers-lab/sessions/2026-05-13T04-13-44-163Z_019e1f8a-6563-769b-8463-8b852ffe2e21.jsonl`

Die relevante Diagnose aus dem Session-Log:
- `configuredTransport: "auto"`
- `eventsEmitted: true`
- `phase: "after_message_stream_start"`
- `errorMessage: "WebSocket error"`

Das grenzt den ursprünglichen Fehler als **mid-stream WebSocket failure im Pi-Codex-Pfad** ein, nicht als bloßen Auth-, Modell- oder Promptfehler.

### 2.2 Phase 2 — Erste falsche Richtungsentscheidung
Nach der ersten Analyse wurde Pi vorübergehend in Richtung eines Codex-spezifischen `SSE-first`-Defaults gepatcht. Diese Entscheidung war technisch nachvollziehbar aus Stabilitätssicht, aber fachlich falsch, weil das eigentliche Ziel nicht „SSE statt WebSocket“, sondern „Pi sauber an Codex CLI angleichen“ war.

Diese falsche Richtungsentscheidung wurde später vollständig zurückgenommen.

### 2.3 Phase 3 — Zielkorrektur
Nach Klärung der Produktabsicht wurde die Richtung festgezogen:
1. Pi-CLI ist der zu fixende Client.
2. OpenAI Codex CLI ist die Referenzvorlage für den Transportpfad.
3. Pi soll **nicht** in ein anderes Produktverhalten abbiegen, sondern einen **Pi-spezifischen Fix/Update** erhalten.

### 2.4 Phase 4 — Lokale Runtime-Änderung in Pi
Nach der Korrektur wurde die lokal installierte Pi-Runtime unter `~/.npm-global/lib/node_modules/...` entsprechend angepasst.

Die finale lokale Runtime-Änderung führte ein eingebautes Setting ein:
- `codexWebsocketsEnabled`

Bedeutung:
- `true` -> Pi bleibt für `openai-codex` WebSocket-capable by default.
- `false` -> Pi nutzt SSE für Codex ohne Custom-Provider-Hack.

### 2.5 Phase 5 — Source-of-truth in lokalen Fork portieren
Da das Pi-Repo lokal noch nicht vorhanden war, wurde ein neuer Fork-/Clone-Pfad aufgebaut:
- Fork/Clone:
  - `/home/jan/gh/pi-mono`
- Branch:
  - `fix/codex-ws-transport-alignment`

Anschließend wurden die Runtime-Änderungen sauber in die TypeScript-Quellen des Forks portiert.

### 2.6 Phase 6 — Web-UI-Checkblocker analysieren und beheben
Parallel zeigte die Repo-Hook-/Check-Kette einen separaten Blocker in `packages/web-ui`. Dieser wurde nicht als Zufallsfehler behandelt, sondern lokal sowie gegen die aktuelle Upstream-Historie untersucht und danach im Fork behoben.

## 3. Final umgesetzte Pi-Codex-Transportänderung

### 3.1 Funktionales Zielbild
Die finale Pi-Änderung implementiert:
- built-in Codex-WebSocket-Capability-Setting
- defaultmäßig WS-capable-Codex-Verhalten
- Pi-native SSE-Opt-out ohne Custom-Provider
- sticky fallback nach Codex-WebSocket-Fehlern
- sicheren Schutz vor Replay nach bereits gestartetem Stream

### 3.2 Finaler eingebauter Setting-Name
- `codexWebsocketsEnabled`

### 3.3 Dateien der finalen Transportimplementierung im Fork
- `packages/coding-agent/src/core/settings-manager.ts`
- `packages/coding-agent/src/core/sdk.ts`
- `packages/coding-agent/src/modes/interactive/interactive-mode.ts`
- `packages/coding-agent/src/modes/interactive/components/settings-selector.ts`
- `packages/agent/src/agent.ts`
- `packages/agent/src/types.ts`
- `packages/ai/src/types.ts`
- `packages/ai/src/providers/openai-codex-responses.ts`
- `packages/coding-agent/docs/settings.md`
- `packages/coding-agent/docs/providers.md`

### 3.4 Commit für den Pi-Codex-Transportfix
- Commit: `2e4eb5d4`
- Message: `fix: align codex transport handling with codex cli`

## 4. Web-UI-Fix der Session

### 4.1 Problem
`packages/web-ui/example/tsconfig.json` referenzierte gebaute `dist/*.d.ts`-Artefakte. In einem frischen Checkout fehlen diese oft, wodurch der Typecheck nicht zuverlässig aus Source funktioniert.

### 4.2 Umgesetzter Fix
`packages/web-ui/example/tsconfig.json` wurde auf source-basierte Workspace-Auflösung zurückgestellt.

### 4.3 Commit für den Web-UI-Fix
- Commit: `44c24a32`
- Message: `fix(web-ui): restore clean example typecheck`

## 5. Session-Dokumentation im Repo

### 5.1 Diese ausführliche Session-Doku
- Pfad:
  - `/home/jan/gh/pi-mono/docs/2026-05-13-session-findings-pi-codex-transport-und-web-ui.md`

### 5.2 Frühere kombinierte Kurz-Doku
- Pfad:
  - `/home/jan/gh/pi-mono/docs/2026-05-13-session-findings-codex-transport-and-web-ui.md`

### 5.3 Commit für die Dokuablage
- Commit: `44e74bf7`
- Message: `docs: capture codex transport and web-ui session findings`

## 6. Validierungsergebnisse der Session

### 6.1 Lokale Pi-Runtime-Validierung
Folgende Artefakte wurden erzeugt:
- `/tmp/pi-codex-ws-default.jsonl`
- `/tmp/pi-codex-ws-disabled.jsonl`
- zusätzliche Zwischenartefakte:
  - `/tmp/pi-codex-sse-default.jsonl`
  - `/tmp/pi-codex-websocket-optin.jsonl`

#### 6.1.1 Default-Run
- Ergebnis: erfolgreich
- Beobachtung:
  - Antwort `OK`
  - kein frischer `WebSocket error`
  - kein frischer `provider_transport_failure`

#### 6.1.2 Disable-WebSockets-Run
- Ergebnis: erfolgreich
- Beobachtung:
  - Antwort `OK`
  - kein frischer `WebSocket error`
  - kein frischer `provider_transport_failure`

### 6.2 Fork-/Repo-Validierung
#### 6.2.1 Pi-Codex-Fix
Die Repo-Hook-Kette war während des frühen Fork-Portings noch teilweise von Fremdfehlern beeinflusst. Die Änderungen wurden deshalb später erneut gegen die lokale Runtime sowie gegen Repo-Surfaces geprüft.

#### 6.2.2 Web-UI-Fix
- `cd /home/jan/gh/pi-mono/packages/web-ui/example && tsc --project tsconfig.json --noEmit --pretty false`
  - erfolgreich
- `cd /home/jan/gh/pi-mono/packages/web-ui && npm run check`
  - erfolgreich

## 7. Wichtige Session-Befunde

### 7.1 Was die Session bestätigt hat
- Pi hatte ein **eigenes** Problem im Codex-Transportpfad.
- Dieses Problem war nicht rein ein OpenAI-Upstream-Thema, auch wenn Upstream ähnliche WS-Fälle kennt.
- Ein SSE-first-Default für Codex wäre fachlich die falsche Richtung gewesen.
- Die richtige Pi-Lösung ist ein **WS-first-Fix/Update** mit eingebautem provider-spezifischem Capability-Schalter.

### 7.2 Was die Session nicht vollständig beweisen konnte
- Ein echter Live-WebSocket-Fehler wurde im finalen WS-first-Stand nicht erneut provoziert.
- Sticky fallback bleibt deshalb in dieser Session auf zwei Fundamenten abgesichert:
  1. Codepfad-Inspektion
  2. historische reale Fehler-Session

### 7.3 Wichtige Qualitätseinschränkung
Es wurde kein wirklich unabhängiger `code-reviewer`-Durchlauf außerhalb der Parent-Session durchgeführt. Das bedeutet:
- Implementierung ist lokal verifiziert und im Fork gesichert.
- Ein zusätzlicher externer Conformance-/Review-Pass wäre für Langzeitpflege weiterhin wertvoll.

## 8. Branch- und Git-Status am Ende der Session

### 8.1 Branch
- `fix/codex-ws-transport-alignment`

### 8.2 Commit-Reihenfolge der Sessionarbeiten
1. `2e4eb5d4 fix: align codex transport handling with codex cli`
2. `44c24a32 fix(web-ui): restore clean example typecheck`
3. `44e74bf7 docs: capture codex transport and web-ui session findings`

### 8.3 Remote-Sync
- Der Branch wurde auf `origin` gepusht.
- Lokaler Branch und `origin/fix/codex-ws-transport-alignment` waren am Ende synchron.

## 9. Reuse-Wert dieser Session

### 9.1 Für den Pi-Codex-Fix
- Die Session dokumentiert, warum `codexWebsocketsEnabled` in Pi sinnvoll ist.
- Sie dokumentiert, warum WS-first die korrekte Richtung ist.
- Sie dokumentiert, warum Pi trotz Codex-CLI-Vorlage keinen Replay nach `after_message_stream_start` einführen sollte.

### 9.2 Für `packages/web-ui`
- Die Session zeigt, dass der `dist/index.d.ts`-abhängige Clean-Checkout-Fehler nicht neu ist.
- Sie dokumentiert, dass die source-basierte Auflösung im Example die richtige Korrekturrichtung ist.

## 10. Empfohlene nächste Schritte
- [ ] Zusätzliche fokussierte automatische Tests für `codexWebsocketsEnabled`, sticky fallback und no-replay ergänzen.
- [ ] Bei Maintainer-Updates am 16./17. den Branch gegen den neuen Refactor-Stand neu prüfen.
- [ ] Falls nötig: die drei Session-Commits gezielt rebasen oder cherry-picken.

## 11. Rollback-Hinweise
- [ ] Für Transportfix nur Commit `2e4eb5d4` revertieren.
- [ ] Für Web-UI-Fix nur Commit `44c24a32` revertieren.
- [ ] Für Session-Doku nur Commit `44e74bf7` revertieren.
- [ ] Für vollständigen Session-Rollback alle drei Commits in umgekehrter Reihenfolge revertieren.
