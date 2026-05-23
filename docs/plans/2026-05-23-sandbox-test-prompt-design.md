# Sandbox Test Prompt Feature

Local-only prompt testing while connected to the Arena server, so participants can interact with their LLM between official challenges.

## Behavior & Lifecycle

The sandbox is available only when `connectionStatus` is `connected` (not during official generation, not when disconnected). It appears as a collapsible section below the official "Resposta Gerada" panel.

**Flow:**
1. User types a prompt and clicks "Enviar Teste" (or hits Enter).
2. The app calls `this.runner.generate()` with the user's prompt — same runner, model, and streaming infrastructure used for challenges.
3. Tokens stream into the sandbox output area with live metrics (tokens, tokens/s, TTFT, duration).
4. No WebSocket messages are sent to the Arena server. This is purely local.

**Auto-cancel on challenge:** If a `challenge` message arrives mid-test, the test generation is immediately aborted. The sandbox output stays visible (not cleared), but the app transitions to official `generating` state. This is done by calling `cancelTest()` at the top of `handleChallenge()`.

**State:** A single `isTestGenerating` flag on the `App` instance, separate from the existing `isGenerating` flag.

## UI Layout

The sandbox section uses the same visual language as the rest of the app with subtle differentiation:

- **Header:** "Sandbox" with a "(teste local)" label.
- **Input row:** Text input + "Enviar Teste" button. Button changes to "Parar" (stop) during test generation.
- **Output area:** Styled like `#token-stream` but with a left border in `--accent-warning` (amber) to signal "unofficial."
- **Metrics row:** Compact display of tokens, tokens/s, TTFT, duration — scoped to the test run.
- **Clear button:** "Limpar" to reset sandbox output.

**Visibility rules:**
- Entire section hidden when disconnected.
- Input and button disabled while an official challenge is generating.
- No new config fields — uses existing runner/model/LLM URL. Defaults: `temperature: 0.8`, `maxTokens: 400`.

## Implementation

All changes in `index.html` (single-file app).

### CSS
- Styles for `#sandbox-panel`: reuses existing section styling, adds left border with `--accent-warning` on the output area.
- Disabled state styling for input/button during official rounds.

### HTML
- New `<section id="sandbox-panel">` after `#output-panel` with prompt input, button, output stream div, and metrics row.

### JavaScript

**`App` class:**
- Add `isTestGenerating` flag.
- Add `testGenerate()` method: calls `this.runner.generate()` with user's prompt, streams to sandbox output, updates sandbox metrics.
- Add `cancelTest()` method: sets `isTestGenerating = false` (token callback already checks the flag).

**`App.handleChallenge()`:**
- Call `this.cancelTest()` at the top — the auto-cancel mechanism.

**`UIController`:**
- Add `updateSandbox()` for sandbox metrics.
- Extend `updateConnectionStatus()` to show/hide sandbox panel and disable input during official generation.

**No changes to `AppState`** — sandbox state is transient, held on `App` instance, DOM updated directly.

**No changes to `ArenaClient`** — no server messages involved.
