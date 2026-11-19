# Punto (was Changeover) — padel scoreboard & side-change coach

## Who you are working with
- The owner is **not a technical person** and builds side projects alone. Explain choices in plain language, avoid jargon, and say what they need to do on their side (e.g. on GitHub or on the iPhone) step by step.
- **Ask before building** a new feature, mockup or big change, especially while ideas are still being discussed. Small, clearly requested fixes can go ahead.
- They play padel themselves and are the main user. Real-world use on court is what matters most.

## What the app is
A padel match app used **mainly on iPhone**, with the phone placed at the court facing the players.
- **Scoreboard**: big team halves with points / games / sets, readable from ~10 m. Shows who serves, from which side (right/left) and who receives.
- **Players & positions**: 4 names, each team has a *drive* (right side) and *revés* (left side) player, plus serve order.
- **Deuce rules** (chosen per match, default **Star point**):
  - Star point (FIP 2026): advantage at 1st and 2nd deuce, single deciding point at the 3rd deuce (5–5 in points); receiving pair chooses who receives.
  - Golden point: single deciding point at 40–40 (3–3).
  - Advantage: classic, no decider.
- **Point input (temporary)**: tap the half of the team that won the point (or ←/→ keys). Undo always available. All scoring goes through one function, `award(team)` — any future input method must call it.
- **Timing**: match, set, game, point lengths; changeover breaks are excluded.
- **Microphone (optional)**: detects ball bounces/strikes (sharp sounds) and attaches a "bounces" count to each point. Glass/racket hits also count — it is a *relative* measure, not an exact shot count.
- **Serve screen**: after every game a full-screen "X serves to Y" message (tap to close; auto-closes after 6 s, but stays until tapped at a change of ends; the clock is paused while it is up, so breaks are not counted as point time).
- **Coach (on demand)**: owner's group stops only between sets, so the coach no longer opens by itself at changeovers — "Coach" button / C key opens it (no timer); it still opens once at match end. Shows one focus tip (e.g. "Protect Luca's serve", "Keep the points short") from hold/break per player, momentum, rally length vs points won, deciding points. Optional "what hurt you most?" tags sharpen the tip.
- **Insights tab**: win chance after every point (Markov model on serve/return points), rally radar, time on court, per-player serve holds and return points won.
- **Home + history**: resume / end & save the current match, New match, history of saved matches with a full report per match (delete possible). Finished matches auto-save; "End" saves unfinished ones (court time ran out).
- **English / Italian**: follows the phone's language, "Italiano / English" button on Home (and L key) overrides it and is remembered. Italian positions: *dritto* / *rovescio*. In code: `tr(en,it)` for dynamic text; static HTML carries `data-it` (innerHTML), `data-it-ph` (placeholder), `data-it-aria` (aria-label), applied by `applyLang()`. **Every new user-facing string needs both languages.**
- Dark/light theme (follows system, toggle remembered), help overlay (`?`), keyboard shortcuts for every action, demo data on first run ("Clear demo data" removes it).

## How it is built (today)
- **One file: `index.html`** (was `changeover.html`; renamed so the GitHub Pages link opens it directly) — HTML, CSS and JavaScript inline, no libraries, no network requests, system fonts only.
- Structure inside the script:
  - **Model (pure functions)**: `replay(match)` rebuilds the whole state (score, server rotation, tiebreaks, stats, timings) from the list of points. Undo and rule changes work by replaying. `winProb`, `rallyInsight`, `coach`, `simulate` (demo data).
  - **View**: `render()` → `renderHome` / `renderBoard` / `renderStats`, plus coach screen, setup sheet (new/edit), report sheet, help.
  - **Input**: `award(w)` (w = 0 your team, 1 rivals), `actions.undo/end/...`, keyboard handler.
  - **Mic**: Web Audio high-pass filter + analyser, adaptive noise floor, sensitivity 1–9.
- **Match data**: `{id, players:[[drive,revés],[drive,revés]], first, order:[i,i], rule:'star'|'golden'|'adv', points:[{w, d (ms), s (bounces, optional)}], notes:[{at, tag}], mark, start, demo?}`.
- **Storage (localStorage)**: `changeover.v2` current match, `changeover.history` saved matches, `changeover.last` last setup, `changeover.theme`, `changeover.sens`, `changeover.lang`. Old saves with a `golden` flag are migrated by `fix()`. A failed write shows a one-time "Not saved" toast; `navigator.storage.persist()` is requested when a match starts; deleting a saved match needs a second tap within 4 s.
- **iPhone storage caveat**: Safari tab and Home Screen app have separate storage, and Safari tabs can be wiped after ~7 days unused → owner should always use the Home Screen icon. The only copy is on the phone.

## Hard-won details (don't break these)
- Serve rotation: team A player → team B player → A partner → B partner. Side: right on even point count, left on odd. Receiver = drive on right, revés on left. Tiebreak: first server 1 point, then 2 each; change ends every 6 points.
- At golden/star deciding point the receiving pair chooses who receives (no fixed receiver highlighted).
- Changeovers: after odd games in a set; also at set end if the set's total is odd.
- iPhone: inputs must be ≥16px font (stops zoom), tap targets ≥44px, safe-area insets, `100dvh`, no Fullscreen API on iPhone (use Add to Home Screen). Wake Lock and microphone need **https**.
- No console errors or warnings, including when the mic is denied or storage is blocked.

## What we have done (history)
1. Explored padel app ideas (shot logger, mic rally counter, voice umpire, etc.).
2. "Shot DNA" (tap-to-log every shot) — dropped: you can't log shots while playing.
3. "Changeover" v1: side-change coach with 2-tap game logging + mic rally counting.
4. v2: point-by-point scoreboard facing the court, names/positions, serve/receiver display, timings, auto coach at changeovers.
5. v3 (current): home page, match history + reports, star/golden/advantage rule selector, iPhone portrait/landscape layouts and touch fixes.
- Tested only in headless Chrome emulating iPhone sizes — **not yet on a real iPhone / Safari**.

## What we will do next (roadmap — confirm order with the owner first)
1. **Host it** on GitHub Pages (https link) so it runs on the iPhone. Public repo `alessiopin1997-ship-it/punto` → https://alessiopin1997-ship-it.github.io/punto/ (gh CLI is installed and logged in; publish = commit + push to `main`).
2. **Test on the real iPhone** (Safari and "Add to Home Screen"): portrait/landscape, touch, wake lock, microphone permission, bounce detection on a real court (tune sensitivity defaults).
3. **Offline use (PWA)**: web app manifest, icons, service worker so it works with poor signal at the court. This means moving from one file to a few files — fine for the product.
4. **Wrist control (agreed plan)**: owner has a **Xiaomi Mi Band 9 + iPhone**. A native band app is not viable (no iPhone install route, Vela interconnect is Android-only), so we use the band's **music controls** via the Media Session API: the page plays a silent loop to become "now playing"; **next track → `award(0)`, previous track → `award(1)`**; **play/pause → undo with confirmation** (1st press sets title "Undo 40–30? Press again", 2nd press within 5 s undoes, else reverts). Track title = compact score, artist = server. Same feature covers earbuds, Apple Watch and ring/button remotes that send media keys. Risks to test on court: iOS dropping "now playing", conflict with the bounce mic, refresh speed of the band title. Optional idea offered: distinct beeps per team/undo.
5. **Backup** (agreed, after the first real on-court test): export all matches to a file (Files / iCloud / email) and import them back.
6. Later ideas (not agreed yet): export/share match reports, stats across matches (per partner/opponent), super tiebreak option for the third set.

## How to work in this repo
- Keep the model logic pure and the code readable; short comments that explain intent.
- After changes, check: new match → score a star point → serve screen after a game → Coach button → end → history report → undo, in portrait and landscape, dark and light, mic denied.
- Commit with clear messages; never lose the owner's saved match history (keep storage keys or migrate them).
