# @zgltyq/pi-ask-user

A fork of [pi-ask-user](https://github.com/edlsh/pi-ask-user) with critical bug fixes and UX improvements for the interactive `ask_user` tool.

## What's fixed in this fork

This fork addresses several issues found in the original extension:

1. **Full-screen mode instead of overlay** — Switches from a floating overlay (`overlay: true`) to a full-screen `custom()` component. This eliminates image overlap issues in terminals using persistent graphics layers (Kitty, Ghostty, WezTerm).

2. **Selection not lost / LLM sees the answer** — Adds a `safeOnDone` guard that prevents `done()` from being called multiple times due to key-repeat, buffered escape sequences, or trailing input events. Previously, a valid selection could be silently overwritten with `null` (cancelled).

3. **Timeout race condition fixed** — Adds a `safeDone` wrapper at the factory level that clears the timeout and removes the abort listener as soon as the user answers. Previously, answering before the timeout expired could still result in the timeout callback overwriting the answer with `null`.

4. **Abort listener memory leak** — The abort signal listener is now explicitly removed when `safeDone` resolves, preventing accumulation across tool calls.

5. **Performance: cached box borders** — `BoxBorderTop` and `BoxBorderBottom` are created once in the constructor and reused, instead of being allocated on every `render()` frame (which happens on every keystroke during search).

6. **Multi-select uses full terminal height** — Removed the hardcoded 10-row cap on `MultiSelectList`. Now dynamically computes available rows from terminal height, same as single-select.

7. **Removed all `as any` casts** — Editor methods (`getText`, `setText`, `focused`) now use the properly typed public API instead of runtime reflection.

8. **`dispose()` cleanup** — Added a `dispose()` method that nulls out all callbacks (`onSubmit`, `onCancel`, `onEnterFreeform`) when the component closes, preventing closure references from leaking.

9. **Number keys work for 10+ options** — Regex changed from `/^[1-9]$/` to `/^[1-9][0-9]?$/` with bounds checking, so options beyond 9 can be selected by typing their number.

10. **Cleaned up unused code** — Removed `ASK_USER_VERSION`, `createRequire`, overlay constants, and the version label from the bottom border (saves one row of vertical space).

## Demo

![ask_user demo](./media/ask-user-demo.gif)

High-quality video: [ask-user-demo.mp4](./media/ask-user-demo.mp4)

## Fork notice

This is a maintained fork of the original [edlsh/pi-ask-user](https://github.com/edlsh/pi-ask-user). All fixes above are applied on top of v0.6.1. If you are using the original package, consider switching to this fork for the stability improvements.

## Features

- **Full-screen interactive UI** — replaces the terminal, no overlay compositing issues with persistent images (e.g. Kitty graphics protocol)
- Searchable single-select option lists with wrapped titles and descriptions
- Responsive split-pane details preview on wide terminals with single-column fallback on narrow terminals
- Multi-select option lists with full terminal height utilization
- Optional freeform responses
- User-toggleable extra context on structured selections
- Context display support
- Pi-TUI-aligned keybinding and editor behavior
- Custom TUI rendering for tool calls and results
- System prompt integration via `promptSnippet` and `promptGuidelines`
- Optional timeout for auto-dismiss in both full-screen and fallback input modes
- Structured `details` on all results for session state reconstruction
- Graceful fallback when interactive UI is unavailable
- Bundled `ask-user` skill for mandatory decision-gating in high-stakes or ambiguous tasks

## Bundled skill: `ask-user`

This package now ships a skill at `skills/ask-user/SKILL.md` that nudges/mandates the agent to use `ask_user` when:

- architectural trade-offs are high impact
- requirements are ambiguous or conflicting
- assumptions would materially change implementation

The skill follows a "decision handshake" flow:

1. Gather evidence and summarize context
2. Ask one focused question via `ask_user`
3. Wait for explicit user choice
4. Confirm the decision, then proceed

See: `skills/ask-user/references/ask-user-skill-extension-spec.md`.

## Install

```bash
pi install npm:pi-ask-user
```

## Tool name

The registered tool name is:

- `ask_user`

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `question` | `string` | *required* | The question to ask the user |
| `context` | `string?` | — | Relevant context summary shown before the question |
| `options` | `(string \| {title, description?})[]?` | `[]` | Multiple-choice options |
| `allowMultiple` | `boolean?` | `false` | Enable multi-select mode |
| `allowFreeform` | `boolean?` | `true` | Add a "Type something" freeform option |
| `allowComment` | `boolean?` | `false` | Expose a user-toggleable extra-context option in the overlay (`ctrl+g` or the toggle row) and collect an optional comment in fallback dialogs |
| `timeout` | `number?` | — | Auto-dismiss after N ms and return `null` if the prompt times out |

## Example usage shape

```json
{
  "question": "Which option should we use?",
  "context": "We are choosing a deploy target.",
  "options": [
    "staging",
    { "title": "production", "description": "Customer-facing" }
  ],
  "allowMultiple": false,
  "allowFreeform": true,
  "allowComment": true
}
```

## Result details

All tool results include a structured `details` object for rendering and session state reconstruction:

```typescript
type AskResponse =
  | { kind: "selection"; selections: string[]; comment?: string }
  | { kind: "freeform"; text: string };

interface AskToolDetails {
  question: string;
  context?: string;
  options: QuestionOption[];
  response: AskResponse | null;
  cancelled: boolean;
}
```

## Changelog

See [CHANGELOG.md](./CHANGELOG.md).