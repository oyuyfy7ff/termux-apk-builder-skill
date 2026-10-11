# recipes.md (companion file of apk.md)

Use when: the Capacitor app needs movable and resizable pieces (images, stickers) or a chat with an AI service. Needed in Phase 3, only for that feature.
Read apk.md first. It holds the rules and the phases. Use the classes of `ui.md` as well.

## Recipes (Capacitor)
Proven in a build and in a desktop browser test. Each one is NOT proven on a real phone until the user confirms it.

### Pieces the user can move and resize (images, stickers)
- Tap selects. Only the selected piece reacts to drag (`touch-action: none` only on the selected one, so the page still scrolls with a finger). Tapping outside, or switching to a drawing mode, unselects.
- Handles: a corner dot to resize (keeps the aspect ratio) and a small button to delete, each with a 48 px touch area. Put them on opposite corners so small pieces do not make them overlap.
- Use pointer events and `setPointerCapture`. After a drag, ignore the click that follows for about 300 ms, or the piece gets selected or deleted by accident.
- Save the position as fractions of the container width (`x`, `w`) and pixels for `y`, plus the aspect ratio. Fractions survive rotation. Give old items that have no saved place a default (full width, stacked) the first time they load.
- Keep the container height equal to the lowest piece, so the text below moves down.

### Chat with an AI service (OpenRouter example)
- The user types the API key and the model name in Settings. They are stored on the device (`localStorage`) and never go in the repo or in the code. This is Option A of `security.md`.
- If `security.md` applies, add `https://openrouter.ai` to `connect-src` in the Content Security Policy.
- Request: `POST https://openrouter.ai/api/v1/chat/completions`, headers `Authorization: Bearer <key>` and `Content-Type: application/json`, body `{ "model": ..., "messages": [...] }`. First version without streaming, with a 90 second limit (`AbortController`).
- Send a short system message and only the last 16 messages. Keep the last 40 messages in `localStorage`.
- Error text for the user, in plain words: 401 key wrong, 402 no credit, 404 model name wrong, 429 too many requests, network failure "cannot connect", timeout "waited too long". On any error remove the failed message from the history and put the typed text back in the box.
- Show the reply as text nodes, never as HTML from the service. Simple bold, bullets and code can be built with DOM nodes.
- Buttons that use the user's own notes (summarize, analyze, search) send only the chosen text, with a length limit, and say which note it came from. A note with no text shows a message instead of an empty request.
- The AI cannot test an outside service with the user's real key. Test with a mocked reply, then ask the user to try one real question.

