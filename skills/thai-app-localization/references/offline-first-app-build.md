# Offline-first Thai app build (React + Vite + TS + Dexie → Capacitor)

Working reference from the Planner Thai build (`C:\Users\Acer\Desktop\Planner-App\app`).
Applies to any small consumer app for Thai users that must work offline + Android-first.

## Stack decision
- **React + Vite + TypeScript + Dexie (IndexedDB)** + Capacitor for Android — 1 codebase → web + iOS + Android.
- Reuse the user's existing web stack (they already run Vite+React+shadcn+Supabase+Capacitor for meekamrai). Do NOT reach for Flutter/React Native unless real native perf is needed — a planner/to-do app is not.
- Verify with Node/npm first; on this machine `node 22` / `npm 12`.

## Local-first architecture (offline-first)
- **Local DB is source of truth** (IndexedDB via Dexie). Sync to server later, when a backend exists. Privacy + offline + speed for Thai users (flaky networks, data-sovereignty).
- IndexedDB (Dexie) is enough for V1; SQLite plugin only if you need real SQL / big data.

## Data model (planner class)
```
tasks      (id, title, note?, due?: 'YYYY-MM-DD', time?: 'HH:MM', priority low|med|high, category, done, createdAt)
habits     (id, name, icon, color, targetPerWeek, createdAt)
habit_logs (id, habitId, date:'YYYY-MM-DD', done)   ← one-tap complete = insert log
```
- **Compute streak/heatmap from `habit_logs` every time** (source of truth); cache into a meta table only if it gets slow.
- Streak with a **grace period** (default 1 missed day) = soft, non-punishing (fits Thai เกรงใจ tone).
- `dayKey(d)` helper: `${y}-${pad(m)}-${pad(d)}` — centralize date-key logic, reuse everywhere.
- Recurring tasks: start with template + occurrence (simple); don't build full RRULE/iCalendar up front.

## Rule-based Thai planner (no-LLM V1)
Instead of an LLM call, a small regex parser handles Thai time/date words and works fully offline:
- Split items on `/[，,\n]+|และ/` (comma/thai-comma/newline, and the word "และ"). **Gotcha:** a character-class `[และ]+` splits into เ/ล/ะ — use the literal word `และ` as an alternation, not inside `[]`.
- Slot hints → time: เช้า→09:00, เที่ยง/กลางวัน/บ่าย→13:00, เย็น→18:00, ค่ำ/ดึก→20:00.
- Day hints → offset: มะรืน=2, พรุ่งนี้=1, default 0.
- Strip the hint keywords from the title; fall back to the full item if stripping empties it.
- Hook point left in the code for a real LLM later (`ponytail:` comment) — swap the parser, keep the output contract.
- Tests: `tsx src/lib/<x>.test.ts` assert-based self-check; add each new test file to the `npm test` script.

## Local notifications
- Web: `Notification` API + `setTimeout` (only fires while the tab is open) — fine for dev.
- Android: `@capacitor/local-notifications`; remember `POST_NOTIFICATIONS`, `SCHEDULE_EXACT_ALARM`, `RECEIVE_BOOT_COMPLETED` permissions in the manifest.
- `toDateTime(due, time)` → `new Date(y, m-1, d, hh, mm)`.

## Pitfall: dev-server port collision (verified on this machine)
- The user runs **Godforge on :5173**. A fresh `vite` picks the next free port (5174+) — **never assume the port**.
- A Hermes `background=true` terminal command must NOT chain `& sleep; curl` — that produced a bash syntax error and the dev server never actually started, so the 200 curl hit Godforge instead.
- **Do:** `npm run dev -- --port 5180 --strictPort` as its own `background=true` call, then `curl localhost:5180` to confirm; open the *correct* port in the browser. Verify the bound port by reading the dev server's own output, not by curling a guess.
