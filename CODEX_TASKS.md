# Codex Task Plan — Solo Leveling Fitness HUD Rebuild

## How to use this file (read this first, every session)

This repo currently contains exactly one file: `index.html`. It is a **minified Vite production build** of a working React app — not source code. Nobody should hand-edit it.

Your job, one session at a time:
1. Read this file top to bottom.
2. Find the **first unchecked task** (`- [ ]`).
3. Do only that task. Do not start the next one, even if you have room left — sessions are kept small on purpose to fit a free-tier usage limit.
4. When it's done and working, change its checkbox to `- [x]` and add a one-line note under it describing what you did (e.g. `- [x] 0.1 Bootstrapped Vite+React+TS project. Old build backed up to reference/index.html.`).
5. Stop. Do not touch later tasks, even to "get ahead."

If a task's acceptance criteria can't be met (missing info, ambiguous requirement), stop, leave the checkbox unchecked, and write a `> BLOCKED:` note under it explaining what's needed instead of guessing.

## Constraints that apply to every task

- **Zero budget.** Every dependency, service, and hosting choice must have a real free tier with no card required for what this app needs. Flag anything that risks that.
- **The developer is learning to code as this is built.** Prefer clear, ordinary code over clever abstractions. Add short comments explaining *why*, not just *what*, especially for game-logic formulas ported from the old build.
- **Target: static hosting on GitHub Pages** (plus free serverless functions once Milestone 8 starts — Supabase Edge Functions).
- **`reference/index.html`** (the old build, moved there in task 0.1) is the source of truth for exact numbers, thresholds, and behavior during the rebuild. When in doubt about what a feature should do, check it there before guessing. Once Milestone 7 is complete and the new app is functionally equivalent, this reference file can be deleted.

---

## Milestone 0 — Bootstrap

- [x] **0.1** Move the existing `index.html` to `reference/index.html`. Initialize a fresh Vite + React + TypeScript project in the repo root (`npm create vite@latest . -- --template react-ts`). Add a `.gitignore` for `node_modules`/`dist`. Verify `npm run dev` shows the default Vite starter page. Do not port any game logic yet.
  - Bootstrapped the Vite React TypeScript app, moved the legacy build to `reference/index.html`, added the required ignore rules, and verified `npm run dev` serves the default starter page.
- [ ] **0.2** Install `framer-motion`, `@tensorflow/tfjs`, `@tensorflow-models/pose-detection`. Configure `vite.config.ts` with the correct `base` path for GitHub Pages. Add a GitHub Actions workflow that builds on push to `main` and deploys `dist/` to the `gh-pages` branch. Verify a build and a real deploy both succeed.
- [ ] **0.3** Extract the visual theme from `reference/index.html`'s embedded `<style>` block: CSS custom properties (`--cyan`, `--bg-deep`, `--red`, `--gold`, etc.) into `src/styles/theme.css`, and the component classes (`.sys-panel`, `.bar-track`, `.btn-sys`, etc.) into `src/styles/app.css`. Import both in `main.tsx`. Verify the page background, fonts, and button styling visually match the old app even with no game components yet.

## Milestone 1 — State foundation

- [ ] **1.1** Create `src/types.ts` with TypeScript interfaces covering the full game state: level, xp/xpMax, hp/hpMax, mp/mpMax, str/agi/vit, pts, streak, the four exercise fields (push/sit/squat/run + their `*Done` flags), dailyCompleted, gold, inventory, equippedItemId, hudTheme, penalty/inLockdown fields, history, titles/notifiedTitles, potions, relapseTokens, urgent/special quest fields, xpBoost fields. Use `reference/index.html`'s state object literal (search for the default state definition) for the exact field list and defaults.
- [ ] **1.2** Build `src/state/GameContext.tsx`: a React Context + `useReducer` holding this state. Implement load-on-mount and save-on-change against `localStorage`, using the same storage key the old app used (`sl_fitness_v3`) so existing saves carry over. Export a `useGame()` hook. No UI yet beyond a provider wrapping the app.

## Milestone 2 — Status HUD

- [ ] **2.1** Build `src/components/StatusPanel.tsx`: level circle with a quest-completion progress ring, HP/MP/XP bars, rank badge (E through S, derived from level), player name, equipped title, and the points/potions/tokens/streak row. Pure presentational, reads from `useGame()`.
- [ ] **2.2** Build `src/components/AttributesPanel.tsx`: STR/AGI/VIT display with a click-to-allocate interaction that spends one stat point per click, including the HP-max/MP-max recompute when VIT changes.

## Milestone 3 — Daily quest

- [ ] **3.1** Build `src/components/DailyQuest.tsx` and a `QuestItem` subcomponent: one row per exercise with a check-toggle, a manual "log reps" number input + submit, and a progress display. Wire to reducer actions `logExercise(type, amount)` and `toggleQuestCheck(type)`.
- [ ] **3.2** Implement the completion reward as a reducer action: when all four exercises hit their target, grant XP, gold, a full HP/MP restore, streak+1, and a history entry. Keep the targets as fixed constants for now (100 push, 100 sit, 100 squat, 10km run) — dynamic targets are Milestone 9.

## Milestone 4 — Penalty & Lockdown

- [ ] **4.1** Implement the day-rollover check: on load, if the stored quest date isn't today and the previous day wasn't completed, apply the penalty (-20 HP, streak reset, penalty timer starts). Build the penalty countdown UI and the punishment-quest submission form (200 push, 200 sit, 10km run to clear it).
- [ ] **4.2** Build the full-screen Lockdown state (triggered at 0 HP, or a repeated missed penalty) that blocks all other UI until the punishment quest is submitted, plus relapse-token consumption as an escape hatch.

## Milestone 5 — Titles, inventory, shop, log

- [ ] **5.1** Titles panel: unlock conditions by level/streak, click to equip, plus a milestones checklist.
- [ ] **5.2** Inventory drawer and shop: HP-restore potion, XP-boost potion, and gear items with XP multipliers, all spending gold.
- [ ] **5.3** Combat log panel rendering the `history` array.

## Milestone 6 — Camera verification

This is the most complex single piece — kept in three parts on purpose.

- [ ] **6.1** Camera + model setup: request `getUserMedia`, load the MoveNet Lightning detector, render the live video with a canvas skeleton overlay. No rep-counting yet — just a reliably rendering skeleton.
- [ ] **6.2** Calibration phase: port the visibility/side-profile alignment scoring from `reference/index.html` exactly (same thresholds), with a progress bar, transitioning to tracking once stable.
- [ ] **6.3** Rep-counting state machine: per-exercise joint-angle thresholds (elbow for push-ups, hip for sit-ups, knee for squats), up/down phase detection, and an on-accept callback wired to `logExercise`.

## Milestone 7 — Polish

- [ ] **7.1** Urgent quests (60-second timed rare-drop events) and special quests (hidden bonus challenges).
- [ ] **7.2** Reward-choice overlay after daily completion, level-up overlay with particle animation, death/retrial flow.
- [ ] **7.3** Sound effects (Web Audio — rep beep, level-up chime, death drone) and HUD theme switching.

**Checkpoint:** by the end of Milestone 7, the rebuilt app should be functionally equivalent to `reference/index.html`, just in maintainable source. `reference/index.html` can now be deleted. Milestones 8+ are genuinely new work, not a port.

## Milestone 8 — Move state off the client (security)

- [ ] **8.1** Create a free Supabase project. Design a minimal schema: `profiles`, `proof_events` (append-only), `daily_log`. Enable row-level security so the client can read its own rows but can never write directly to `proof_events`.
- [ ] **8.2** Write a Supabase Edge Function that accepts a verified rep submission (exercise type, count, session duration), rejects implausible rep-rates, and writes the authoritative `proof_events` row plus recomputed derived state (trust score / XP / streak).
- [ ] **8.3** Swap `logExercise` and the daily-completion action to call this function instead of writing to `localStorage`. Local state becomes a read-through cache of the last known server response, never a source of truth.

## Milestone 9 — Jarvis (AI goal decomposition)

- [ ] **9.1** Add a goal-input UI (goal text + horizon) and a `daily_log` write every day, not just completed ones — this is the throughput signal Jarvis needs.
- [ ] **9.2** Write a Supabase Edge Function that calls the Gemini API (key stored server-side only) with the goal, current level/stats, and the recent `daily_log`, returning this week's four targets as fixed-shape JSON — no free text to parse.
- [ ] **9.3** Replace the fixed 100/100/100/10 constants throughout the UI with these dynamic targets, and implement the plain-code weekly adaptive rule: raise targets ~10% if completion rate has been above ~90% for the week, ease off if it's been below ~50%, hold steady otherwise.
