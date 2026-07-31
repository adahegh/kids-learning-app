# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

"Learn & Play" — a kids' learning app (writing, math, reading games with an aquarium reward system). Two self-contained HTML files, zero dependencies, no build step, no tests, no lint:
- `index.html` — parent-facing landing page (what gets shared; explains the app, links to the game).
- `play.html` — the app itself: all CSS in one `<style>` block, all JS in one `<script>` block. Tabs (nav order): Reading (Flashcards / Star Catcher / Word Pop), Math (Plus / Minus / Number Bonds / 123 counting mode — Next Number, Hundred Chart, Blast Off backwards-counting, prefix `ct*`, with bilingual English+Mandarin number audio via `ctSpeak()`/`numToZh()` — toggle persisted as `gs.zhAudio`), Writing (upper / lower / numbers / teens tracing, plus a Flip It sub-tab for letter-reversal practice: Trace It / Spot It / Sound Sort), Aquarium.

Deployed via **GitHub Pages** from `main` (repo `adahegh/kids-learning-app`) — pushing to `main` publishes. Both pages carry a GoatCounter analytics script tag before `</body>` (site code `adahejj`; cookie-free pageview counts; ignores localhost). All game state persists to the localStorage key `learnplay_gs`.

## Running

No build. Serve statically and open in a browser:

```sh
python3 -m http.server 4599   # port from .claude/launch.json
```

Target device is an iPad/touch screen (apple-mobile-web-app meta tags, touch + mouse event handlers, `speechSynthesis` for word audio). Test touch interactions when changing canvas or bubble games.

## Architecture (inside play.html)

**Screens & navigation.** Each game is a `<div class="screen" id="screen-*">`; `switchMain(tab)` toggles `.active` and runs per-screen setup (canvas sizing, render). Bottom nav tabs call it.

**Per-game namespaces.** Globals/functions are prefixed by game, e.g.: `w*` (writing), `m*` (math), `r*` (reading flashcards), `wp*` (Word Pop), `sc*` (Star Catcher), `ft*`/`fs*`/`fso*` (Flip It), plus shared `gs*`/aquarium code. Keep new code in the owning prefix.

**Shared gamification layer** (bottom of the script): games report progress only through two calls — `earnXP(amount)` and `tickDaily(game, amount)`. Every 25 XP (`XP_PER_FISH`) mints one fish via `addFish()`; the full-screen reward popup fires **only** when a new `FISH_ROSTER` species unlocks (thresholds are total-fish counts) — keep ordinary progress quiet. Daily-goal bars (`DAILY_TARGETS`), streaks, and milestone overlays hang off the same two calls. Persistent state lives in the `gs` object (`gsSave()`/`gsLoad()`); `gsSave()` also syncs the out-of-`gs` globals (`fishEarned`, `rMastered`, `rCorrectCounts`, `wpBest`) and `gsLoad()` rehydrates them — new persistent state must be wired into both.

**Writing/tracing games.** Letter stroke data lives in `STROKES` (Flip It reuses it, adding mnemonics in `FLIP_ITEMS`/`FLIP_LIST`) as arrays of strokes, each an array of `[x,y]` points in a **100×100 coordinate space**, scaled to canvas size by `sc()`. Two stacked canvases: guide (below) and drawing (above). Stroke acceptance is intentionally lenient — `wScore()` blends direction (50%), endpoint proximity (35%), and length ratio (15%) with a 0.35 pass threshold; keep it forgiving, the user is a young child.

**Reading.** Word lists in `R_WORDS` are the official Dolch levels (`prek` Pre-Primer 40, `k` Primer 52, `g1` First Grade 41). A sight word is "mastered" after 3 correct answers (`rCorrectCounts`) — that feeds the "Words I know" chips and a +10 XP bonus, but does **not** mint fish directly (fish come only from XP). The flashcards' "Still Learning" pill filters to `rStillLearning`, a session-scoped set of words the child marked "😅 Still learning".

## Conventions

- Kid-facing UX: large touch targets, emoji, cheerful feedback messages, no reading required to navigate. Wrong answers reset streaks but always allow retry.
- Code style is dense single-line functions with terse names; match it rather than reformatting.
