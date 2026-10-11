# ui.md (companion file of apk.md)

Use when: any screen, color or layout work. Needed in Phase 0 (to pick hue, character and mode) and for all screen work from Phase 1.
Read apk.md first. It holds the rules and the phases. This file adds detail and is not complete alone.

## UI rules (any kind of app, a real phone app that also looks designed)
These rules are general. They do not depend on the kind of app: the look comes from a theme generator (one hue number makes the whole palette), a few "character" switches, a set of components, and screen recipes picked by what the app shows. Capacitor is covered in detail here; the Flutter and Kotlin look notes are in `flutter.md` and `kotlin.md`.

### How the design work flows
- Phase 0: ask for the screens (list them). Choose the look yourself and confirm it in one line: hue (by mood, see the hue guide), character, mode. Example: "Look: calm (blue-violet, soft corners, follows the phone). Change?" Write the screens and the look in `PROGRESS.md`. Do not invent extra screens.
- Phase 1 and 2: `www/style.css` is Tokens + Theme generator + Components, copied exactly, with only the hue, saturation and character lines changed. No icons, no animation yet.
- Phase 3: build screens from the recipes. Text labels only.
- Phase 4: add icons, motion and polish, then check contrast on the phone in light and in dark.
- Unknown kind of app: do not look for an exact match. Name what the app shows (a list, pictures, a conversation, numbers, a canvas, something to edit), take the recipe for that shape, pick hue and character by mood, and build any missing piece only from tokens and existing classes (see "A piece that does not exist yet").

### Usability rules (always)
- One main action per screen. Big and clear. No tiny links as the only way to do something.
- Touch targets at least 48 px high and wide. Space between buttons at least 8 px.
- Text at least 16 px, except small notes (14 px) and labels under navigation icons (13 px). Input fields at least 16 px (smaller sizes make the page zoom on focus).
- Never rely on hover. Everything must work with a tap.
- Every list needs an empty state (a short line such as "No items yet"). Every action that can fail shows a short error line.
- Thai text: use the system font (`font-family: system-ui, sans-serif`). Do not load fonts from the internet (the app must work offline). Use `line-height: 1.5` for body text and at least `1.35` for headings so Thai marks are not cut.
- Safe areas (notch, status bar, gesture bar): keep content clear of them, see the CSS below.
- Dark mode: all colors are CSS variables switched by `prefers-color-scheme`. Never hard-code a color inside a component (the only exception: overlays on top of video, camera, canvas or images, where the background is unknown).
- Keep the error box script from the Capacitor template while developing. Remove it only at release.
- Do not add animations, icons or images before the first green build. Icons and motion come in Phase 4 (see Icons and Motion).
- After every UI change, tell the user exactly what to look at on the phone ("the Add button is blue and sits under the field"), and ask for a screenshot if something looks wrong.

### Design rules (these make it look designed, not default)
- Spacing comes only from the scale 4, 8, 12, 16, 24, 32 (`var(--s1)` to `var(--s6)`). Screen edge padding 16. Related items 8 apart, separate groups 24 apart.
- Three main text levels: title (28, or 20 in the app bar), body (16), small muted (14). The only other sizes are section headings (`h2` 22, `h3` 18), big numbers (`.stat` 28) and navigation labels (13). Two weights only: 400 and 600 (700 with the `bold` character). Align text left. Never center a long paragraph.
- Four neutral colors (`--bg`, `--surface`, `--text`, `--muted`) plus exactly one accent (`--main`) for the primary button, the selected item and links. `--danger` and `--ok` carry meaning only (error, gain, success).
- One filled primary button per screen. Other actions are `soft` or `ghost`. A destructive action is `danger` (outlined), never a big red filled button.
- One corner radius for cards and buttons (`--radius`), a smaller one for inputs (`--radius-sm`). No random radii.
- Group with space first, then with a surface. Do not draw a border around everything, and do not put a card inside a card.
- Depth: light mode uses the soft shadow on cards. Dark mode has no shadows; raised things use `--surface2`.
- Do not: pure black on pure white, a second accent color, gradients or glow without a reason, gray text on a gray surface, icons smaller than 24 px, long lines of full-width text, emoji (see Style rules).
- Light only: delete the dark blocks. Dark only: delete the light blocks and remove the `@media (prefers-color-scheme: dark)` wrapper from the dark ones, then add `<meta name="color-scheme" content="dark">` to `index.html`.

### Tokens (put first in `www/style.css`)
```css
:root {
  --s1: 4px;
  --s2: 8px;
  --s3: 12px;
  --s4: 16px;
  --s5: 24px;
  --s6: 32px;
  --radius: 14px;
  --radius-sm: 10px;
  --row-h: 56px;
  --head-weight: 600;
  --shadow: 0 1px 2px #0000000d, 0 4px 12px #0000000d;
  --font: system-ui, sans-serif;
  --font-head: var(--font);
  --bar-h: 56px;
  --nav-h: 64px;
}
@media (prefers-color-scheme: dark) {
  :root { --shadow: none; }
}
```

### Theme generator (put second: any app, any color)
One hue number makes the whole palette, in light and in dark. The lightness values are fixed, so text contrast is at least 4.5 to 1 for every hue from 0 to 360 and every `--sat` from 0.2 to 1.4 (checked by calculation for all of them). Edit only `--hue` and `--sat`.
- `--hue`: 0 to 360, choose from the hue guide below.
- `--sat`: color strength. 1 normal, 0.5 quiet, 0.2 almost gray (serious tools), 1.4 vivid (games, kids). Stay between 0.2 and 1.4.
- The first block is a fixed blue fallback for an old Android WebView that cannot read `oklch()`. Never edit it.
```css
/* fallback (old WebView) */
:root {
  --bg: #f6f8fe;
  --surface: #ffffff;
  --surface2: #e8effc;
  --text: #161b24;
  --muted: #555e6f;
  --line: #d9dee8;
  --main: #315dd4;
  --on-main: #ffffff;
  --danger: #b7191c;
  --ok: #137738;
}
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #0c0f16;
    --surface: #141822;
    --surface2: #1e2430;
    --text: #e8ebf2;
    --muted: #9da5b5;
    --line: #2a303d;
    --main: #8bafff;
    --on-main: #0f1624;
    --danger: #f47b74;
    --ok: #76cf8a;
  }
}

/* generator */
@supports (color: oklch(50% 0.1 200)) {
  :root {
    --hue: 265;
    --sat: 1;
    --bg: oklch(98% calc(0.008 * var(--sat)) var(--hue));
    --surface: oklch(100% 0 0);
    --surface2: oklch(95% calc(0.02 * var(--sat)) var(--hue));
    --text: oklch(22% calc(0.02 * var(--sat)) var(--hue));
    --muted: oklch(48% calc(0.03 * var(--sat)) var(--hue));
    --line: oklch(90% calc(0.015 * var(--sat)) var(--hue));
    --main: oklch(52% calc(0.19 * var(--sat)) var(--hue));
    --on-main: oklch(100% 0 0);
    --danger: oklch(50% 0.19 27);
    --ok: oklch(50% 0.13 150);
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: oklch(17% calc(0.015 * var(--sat)) var(--hue));
      --surface: oklch(21% calc(0.02 * var(--sat)) var(--hue));
      --surface2: oklch(26% calc(0.025 * var(--sat)) var(--hue));
      --text: oklch(94% calc(0.01 * var(--sat)) var(--hue));
      --muted: oklch(72% calc(0.025 * var(--sat)) var(--hue));
      --line: oklch(31% calc(0.025 * var(--sat)) var(--hue));
      --main: oklch(76% calc(0.14 * var(--sat)) var(--hue));
      --on-main: oklch(20% 0.03 var(--hue));
      --danger: oklch(72% 0.15 25);
      --ok: oklch(78% 0.13 150);
    }
  }
}
```
Hue guide (pick by mood, any number between also works). The last column is the light `--main` color as hex, used as the Flutter seed and the Kotlin accent:

| Hue | Mood | Light main |
|---|---|---|
| 25 (red) | energy, urgency, alerts, food | #be222a |
| 55 (orange) | warm, friendly, food, social | #9f5100 |
| 95 (yellow) | sunny, playful, creative (olive tone) | #7c6700 |
| 150 (green) | growth, health, nature, money | #007f38 |
| 195 (teal) | calm, clean, tech | #007879 |
| 250 (blue) | trust, work, information | #006bb9 |
| 285 (violet) | creative, media, premium | #634ecf |
| 350 (pink) | love, care, fashion | #b22578 |

- Almost gray (`--sat: 0.2`, hue 260): serious, neutral, admin or utility apps.
- Flutter and Kotlin need one hex per look. For the hues 25, 55, 95, 150, 195, 250, 285 and 350 use the row of the table. For the other hues of the starting points take the nearest row (10 red, 70 orange, 290 violet), and for calm and plain (265, 260) use `#315DD4` light and `#8BAFFF` dark, as in the Flutter and Kotlin examples.
- The user gives an exact brand color: keep the generator for the neutrals, then add two lines after it, in both light and dark: `--main: <their hex>;` and `--on-main: #ffffff;` (or `#111111` when their color is light). Check that `--on-main` is readable on `--main`.

### Character (the feel, independent of color)
Add the lines of the chosen characters after the Tokens block (they override it). Use one or two, not all:

| Character | Lines | Feel |
|---|---|---|
| soft (default) | nothing to add | friendly, rounded |
| sharp | `--radius: 6px; --radius-sm: 4px;` | precise, technical |
| round | `--radius: 20px; --radius-sm: 14px;` | warm, playful |
| pill | `--radius: 28px; --radius-sm: 20px;` | toy-like, bold |
| flat | `--shadow: none;` | quiet, border only |
| compact | `--row-h: 48px;` | dense lists, more on screen |
| roomy | `--row-h: 64px;` | calm, touch friendly |
| serif | `--font-head: serif;` | reading, journal |
| mono | `--font-head: ui-monospace, monospace;` | tools, terminal look (English text only) |
| bold | `--head-weight: 700;` | strong, loud |

Starting points (only examples of hue, saturation and characters; any other mix is fine):

| Name | Typical use | Hue | Sat | Characters |
|---|---|---|---|---|
| calm | tools, notes, to-do | 265 | 1 | soft |
| fresh | health, habits, food | 150 | 0.9 | round |
| warm | social, chat, love, greetings | 10 | 1 | round |
| money | finance, stats, admin | 195 | 0.8 | sharp, flat, compact |
| night | music, video, camera, gallery | 290 | 1 | dark only |
| paper | reading, learning, journal | 70 | 0.5 | serif, roomy |
| play | games, kids | 285 | 1.4 | pill, bold |
| terminal | developer tools, hacker toys | 150 | 0.8 | dark only, mono, sharp, flat |
| plain | anything serious or unknown | 260 | 0.2 | soft |

For numbers (finance, stats, scores): put `class="num"` on every number so digits line up, use `pos` and `neg` for gain and loss, and always add a sign (+ or -), never color alone.

### Components (put third in `www/style.css`; uses only variables)
```css
* { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
html, body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font-family: var(--font);
  font-size: 16px;
  line-height: 1.5;
}
body {
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  padding: 0 env(safe-area-inset-right) 0 env(safe-area-inset-left);
  touch-action: manipulation;
}
h1, h2, h3 { margin: 0; font-family: var(--font-head); font-weight: var(--head-weight); line-height: 1.35; }
h1 { font-size: 28px; }
h2 { font-size: 22px; }
h3 { font-size: 18px; }
p { margin: 0; }
.muted { color: var(--muted); }
.small { font-size: 14px; }
.num { font-variant-numeric: tabular-nums; }
.pos { color: var(--ok); }
.neg { color: var(--danger); }
.error { color: var(--danger); font-size: 14px; }

/* layout */
.appbar {
  position: sticky;
  top: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  gap: var(--s2);
  min-height: calc(var(--bar-h) + env(safe-area-inset-top));
  padding: env(safe-area-inset-top) var(--s4) 0;
  background: var(--bg);
  border-bottom: 1px solid var(--line);
}
.appbar h1 { flex: 1; font-size: 20px; }
.screen {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--s4);
  padding: var(--s4);
}
.screen.no-bar { padding-top: calc(var(--s4) + env(safe-area-inset-top)); }
.grid2 { display: grid; grid-template-columns: 1fr 1fr; gap: var(--s3); }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(140px, 1fr)); gap: var(--s3); }
.toolbar {
  display: flex;
  gap: var(--s1);
  overflow-x: auto;
  padding: var(--s2) var(--s4);
  border-bottom: 1px solid var(--line);
}
.toolbar > * { flex: none; }

/* surfaces */
.card {
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: var(--radius);
  padding: var(--s4);
  box-shadow: var(--shadow);
}
.stat { font-size: 28px; font-weight: 600; line-height: 1.2; }
.list { display: flex; flex-direction: column; gap: var(--s2); margin: 0; padding: 0; list-style: none; }
.row {
  display: flex;
  align-items: center;
  gap: var(--s3);
  min-height: var(--row-h);
  padding: var(--s2) var(--s4);
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: var(--radius);
}
.row .grow { flex: 1; min-width: 0; }
label.row { font-size: 16px; font-weight: 400; }  /* settings row: body text, not the 14 px form label */
.row:active { background: var(--surface2); }
.empty { padding: var(--s6) var(--s4); text-align: center; color: var(--muted); }
.tile { aspect-ratio: 1; overflow: hidden; border-radius: var(--radius-sm); background: var(--surface2); }
.tile img, .tile video { width: 100%; height: 100%; object-fit: cover; display: block; }
.avatar {
  flex: none;
  width: 40px;
  height: 40px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: var(--surface2);
  font-weight: 600;
}
.chip, .badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 24px;
  padding: 0 var(--s3);
  border-radius: 999px;
  background: var(--surface2);
  font-size: 14px;
}
.chip.on, .badge { background: var(--main); color: var(--on-main); }
.progress { height: 8px; border-radius: 999px; background: var(--surface2); overflow: hidden; }
.progress > span { display: block; height: 100%; background: var(--main); }

/* buttons: the plain button element is the primary button */
button, .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--s2);
  min-height: 48px;
  min-width: 48px;
  padding: 0 var(--s5);
  font: inherit;
  font-weight: 600;
  border: 0;
  border-radius: var(--radius);
  background: var(--main);
  color: var(--on-main);
}
button:active, .btn:active { filter: brightness(0.92); }
button:disabled { opacity: 0.5; }
.block { width: 100%; }
.soft { background: var(--surface2); color: var(--text); }
.ghost { background: transparent; color: var(--main); border: 1px solid var(--line); }
.danger { background: transparent; color: var(--danger); border: 1px solid var(--danger); }
.icon-btn { width: 48px; padding: 0; background: transparent; color: var(--text); }
.fab {
  position: fixed;
  right: calc(var(--s4) + env(safe-area-inset-right));
  bottom: calc(var(--nav-h) + var(--s4) + env(safe-area-inset-bottom));
  width: 56px;
  height: 56px;
  padding: 0;
  border-radius: 50%;
  box-shadow: var(--shadow);
}
.fab.no-nav { bottom: calc(var(--s4) + env(safe-area-inset-bottom)); }
.tabs { display: flex; gap: var(--s1); padding: var(--s1); background: var(--surface2); border-radius: var(--radius); }
.tabs button {
  flex: 1;
  padding: 0 var(--s2);
  font-weight: 400;
  background: transparent;
  color: var(--muted);
  border-radius: var(--radius-sm);
}
.tabs button[aria-selected="true"] { background: var(--surface); color: var(--text); font-weight: 600; box-shadow: var(--shadow); }

/* forms */
.field { display: flex; flex-direction: column; gap: var(--s1); }
label { font-size: 14px; font-weight: 600; }
input, textarea, select {
  width: 100%;
  min-height: 48px;
  padding: var(--s3) var(--s4);
  font: inherit;
  font-size: 16px;
  color: var(--text);
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: var(--radius-sm);
}
input:focus, textarea:focus, select:focus { outline: 2px solid var(--main); outline-offset: 1px; }
::placeholder { color: var(--muted); }
.switch {
  -webkit-appearance: none;
  appearance: none;
  position: relative;
  flex: none;
  width: 52px;
  min-height: 32px;
  height: 32px;
  padding: 0;
  border: 0;
  border-radius: 999px;
  background: var(--muted);
}
.switch::after {
  content: "";
  position: absolute;
  top: 4px;
  left: 4px;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: var(--surface);
}
.switch:checked { background: var(--main); }
.switch:checked::after { left: 24px; }

/* bottom navigation: 3 items is best, 5 at most */
.nav {
  position: sticky;
  bottom: 0;
  z-index: 10;
  display: flex;
  background: var(--surface);
  border-top: 1px solid var(--line);
  padding-bottom: env(safe-area-inset-bottom);
}
.nav button {
  flex: 1;
  flex-direction: column;
  gap: 2px;
  min-height: var(--nav-h);
  padding: var(--s2) 0;
  font-size: 13px;
  font-weight: 400;
  background: transparent;
  color: var(--muted);
  border-radius: 0;
}
.nav button[aria-current="page"] { color: var(--main); font-weight: 600; }

/* conversation */
.chat { display: flex; flex-direction: column; gap: var(--s2); }
.bubble {
  align-self: flex-start;
  max-width: 80%;
  padding: var(--s2) var(--s3);
  border-radius: var(--radius);
  background: var(--surface2);
}
.bubble.me { align-self: flex-end; background: var(--main); color: var(--on-main); }
.composer {
  position: sticky;
  bottom: 0;
  display: flex;
  gap: var(--s2);
  padding: var(--s2) var(--s4) calc(var(--s2) + env(safe-area-inset-bottom));
  background: var(--bg);
  border-top: 1px solid var(--line);
}

/* dialog from the bottom */
.scrim {
  position: fixed;
  inset: 0;
  z-index: 20;
  display: flex;
  align-items: flex-end;
  background: #00000066;
}
.scrim[hidden] { display: none; }
.sheet {
  width: 100%;
  max-height: 85dvh;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: var(--s4);
  padding: var(--s5) var(--s4) calc(var(--s5) + env(safe-area-inset-bottom));
  background: var(--surface);
  border-radius: var(--radius) var(--radius) 0 0;
}

/* full screen content: game, camera, map, video, canvas */
.stage { position: fixed; inset: 0; background: var(--bg); }
.stage canvas, .stage video { display: block; width: 100%; height: 100%; }
.hud {
  position: absolute;
  top: calc(var(--s4) + env(safe-area-inset-top));
  left: var(--s4);
  right: var(--s4);
  display: flex;
  justify-content: space-between;
  gap: var(--s2);
  pointer-events: none;
}
.hud > * { pointer-events: auto; }
.controls {
  position: absolute;
  left: var(--s4);
  right: var(--s4);
  bottom: calc(var(--s4) + env(safe-area-inset-bottom));
  display: flex;
  justify-content: center;
  gap: var(--s3);
}
.pill { padding: var(--s2) var(--s4); border-radius: 999px; background: #000000a6; color: #ffffff; }

/* development error box (remove at release) */
#err { color: var(--danger); white-space: pre-wrap; text-align: left; font-size: 14px; }
```
Chips and badges only show a status. A tappable filter is a `soft` button, so the 48 px target rule holds. Controls on a `.stage` use `icon-btn` or `.pill` (white text on a dark overlay) so they stay readable over any content.

### A piece that does not exist yet
When the app needs something the classes above do not cover (a calendar grid, a timer ring, a card deck, a map marker label):
1. Build it only from tokens: spacing from the scale, `--radius` or `--radius-sm`, colors from variables, targets at least 48 px.
2. Add its CSS at the end of the Components part of `www/style.css`, with a one-line comment.
3. Name it in `PROGRESS.md` under the screen that uses it.
4. If it needs a color that has no variable, use `--main`, `--surface2` or `--muted`. Do not add a new accent color.

### Screen recipes (Capacitor markup)
Pick the recipe by what the app shows:

| The app shows | Use |
|---|---|
| a list of things | list screen |
| pictures, videos, products | `grid` of `tile` items, tap opens a detail screen |
| a conversation or a feed | `chat` with `bubble` items and a `composer`, or a `list` of `card` posts with an `avatar` |
| numbers, charts, status | dashboard |
| a form or settings | form screen or settings list |
| a game, camera, map, video, canvas | full screen `stage` with a `hud` and `controls`, no app bar |
| something to edit (text, image, timeline) | main area with a `toolbar`, edits open in a `sheet` |
| a first-run explanation | at most 3 screens, each one `h1`, one `muted` line, one `block` button |
| something not in this table | the closest recipe plus "A piece that does not exist yet" |

Example of a list screen (labels only, icons come in Phase 4):
```html
<header class="appbar"><h1>Tasks</h1></header>
<main class="screen">
  <div class="field">
    <label for="title">New task</label>
    <input id="title" type="text" autocomplete="off">
    <p class="error" id="title-err" hidden>Type something first.</p>
  </div>
  <button type="button" class="block" id="add">Add</button>
  <ul class="list" id="list"></ul>
  <p class="empty" id="empty">No tasks yet</p>
</main>
```
Example of a full screen game or camera screen:
```html
<div class="stage">
  <canvas id="view"></canvas>
  <div class="hud"><span class="pill num" id="score">0</span></div>
  <div class="controls"><button type="button" id="action">Start</button></div>
</div>
```
- List screen: app bar with the title, an optional search field, a `list` of `row` items (title at 16 px, a `muted small` second line), an empty state, and one add action (floating button or a full-width button).
- Form screen: app bar with a back button, one `field` per input with the label above, the error line under the field, one `block` button as the last item. Set the right `type` and `inputmode` (number, email, tel) so the keyboard fits.
- Detail screen: `h1` title, one `muted small` line of facts, a `card` with the content, at most two actions (one primary, one `soft`).
- Dashboard: a `grid2` of two `card` items (a `muted small` label over a `stat` number), a `progress` bar where a goal exists, then a `list` below.
- Settings: a `list` of `label class="row"` items (the `label.row` rule keeps them at 16 px, normal weight), the label text on the left, a `switch` or a chevron icon on the right, so the whole row is the tap target. Group names are `muted small` lines.
- Conversation: newest message at the bottom, `bubble.me` for the user, the `composer` sticks to the bottom with an input and one button.
- Editor: the `toolbar` scrolls sideways inside its own box, the page itself never scrolls sideways. Choices (size, color, format) open in a `sheet`.
- Dialog: a `scrim` with a `sheet`. Put a short title, one line of text, then a primary and a `soft` button. A confirmation for a destructive action uses `danger`.
- Navigation: use the `nav` bar when the app has 2 to 5 top-level screens, or `tabs` when the screens share one topic. More than that: put the extra screens in a settings list.
- A screen with no app bar gets `class="screen no-bar"` so content clears the status bar.
- Loading: a `muted` line "Loading". No spinners before Phase 4.

### Icons (Phase 4)
Inline SVG line icons only, never emoji and never an icon font from the internet. All use the same 24 by 24 grid and 2 px stroke:
```css
.ic {
  width: 24px;
  height: 24px;
  fill: none;
  stroke: currentColor;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
}
```
```html
<button type="button" class="icon-btn" aria-label="Add">
  <svg class="ic" viewBox="0 0 24 24"><path d="M12 5v14M5 12h14"/></svg>
</button>
```
Path data for the `d` attribute (checked by drawing them; do not invent other paths without checking):
- plus: `M12 5v14M5 12h14`
- check: `M20 6L9 17l-5-5`
- close: `M18 6L6 18M6 6l12 12`
- back: `M19 12H5M12 19l-7-7 7-7`
- chevron: `M9 18l6-6-6-6`
- menu: `M4 6h16M4 12h16M4 18h16`
- search: `M18 11a7 7 0 1 1-14 0a7 7 0 0 1 14 0zM21 21l-4.3-4.3`
- trash: `M4 7h16M10 11v6M14 11v6M6 7l1 13h10l1-13M9 7V4h6v3`
- home: `M3 11l9-8 9 8M5 10v10h14V10`
- user: `M16 8a4 4 0 1 1-8 0a4 4 0 0 1 8 0zM4 21c0-4 4-6 8-6s8 2 8 6`
- edit: `M4 20h4L19 9l-4-4L4 16v4zM13 7l4 4`
- heart: `M12 20s-7-4.6-7-10a4 4 0 0 1 7-2.5A4 4 0 0 1 19 10c0 5.4-7 10-7 10z`
- star: `M12 2l3.1 6.3 6.9 1-5 4.9 1.2 6.9L12 17.8l-6.2 3.3L7 14.2 2 9.3l6.9-1L12 2z`

- An icon-only button always has an `aria-label`. Every navigation item has an icon and a text label.
- A delete action uses the `trash` icon in an `icon-btn` and asks for confirmation when the data cannot be recovered.
- An icon that is not in this list: build it from simple lines and circles on the same grid, put it in a test build and ask the user for a screenshot, or use a text label instead.

### Motion (Phase 4)
Short and quiet: 150 to 200 ms, only opacity and transform, and respect the phone setting.
```css
@media (prefers-reduced-motion: no-preference) {
  button, .btn, .row { transition: background-color 0.15s, filter 0.15s, transform 0.15s; }
  button:active, .btn:active { transform: scale(0.97); }
  .screen { animation: rise 0.2s ease-out; }
}
@keyframes rise {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: none; }
}
```

### Capacitor setup
Add `viewport-fit=cover` to the viewport meta tag in `index.html`. The CSS above already keeps the app bar and the navigation bar clear of the status bar and the gesture bar. These insets are the part that is not proven on a real phone yet: in the first build ask the user for a screenshot of the top and the bottom edge.
