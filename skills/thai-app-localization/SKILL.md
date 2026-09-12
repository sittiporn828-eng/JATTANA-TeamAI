---
name: thai-app-localization
description: Use when building apps for Thai users.
version: 1.0.0
author: hermes
license: internal
metadata:
  tags: [thai, localization, i18n, ux, typography, nlp]
  related_skills: [localized-document-generation]
---

# Thai App Localization

## When to Use
Load this whenever building or localizing any app/UI for Thai end users — writing Thai copy, choosing Thai fonts, designing Thai UX, or adding Thai-aware AI/NLP features. Also useful for Thai-market competitive/product analysis.

Making an app genuinely Thai, not just translated. Applies to consumer apps for the Thai market (Planner Thai, meekamrai, MediLINE, etc.). Root reference reports: `C:\Users\Acer\Desktop\Planner-App\research\thai-planner-app-research.md` and `planner-app-ux-ui.md`.

## Correct Thai for apps (ราชบัณฑิตยสภา)
- Follow the Royal Institute (ราชบัณฑิตยสภา) for spelling, transliteration (ทับศัพท์), and terminology.
- Common misspellings to avoid in UI copy:
  - แอปพลิเคชัน (NOT แอพพลิเคชั่น)
  - เบราว์เซอร์ (NOT บราวเซอร์ / เบราเซอร์)
  - คลิก (NOT คลิ๊ก)
  - อินเทอร์เน็ต, ดิจิทัล, ไฟล์
- Long-used loanwords keep dictionary form (ช็อกโกแลต, ก๊าซ). Transliteration lookup: https://transliteration.orst.go.th/

## 5 Thai language registers — match to UI context
1. **พิธีการ** (grand/royal, ราชาศัพท์) — avoid in consumer UI
2. **ทางการ** (formal) — Terms, privacy, financial confirmations
3. **กึ่งทางการ** (semi-formal) — feature descriptions, onboarding copy
4. **ไม่เป็นทางการ/สนทนา** (conversational) — main UI, buttons, status
5. **กันเอง** (intimate) — short notifications
- Rule of thumb: buttons/status = conversational but polite; notifications = semi-formal; legal = formal.
- Use ครับ/ค่ะ politeness particles. Avoid bossy tone — Thais value เกรงใจ (consideration); a gentle, soft tone resonates.

## Thai UX/UI
- **Android-first** (~80%+ of Thai users), low-end devices. Zero-setup onboarding — feature-based onboarding is perceived as marketing (Nielsen Norman). Get the user to a first win in ~60s.
- **Thai culture templates**: Thai holidays, วันพระ, the salary/เงินเดือน cycle, ทำบุญ/งานบุญ/วัด, festivals (สงกรานต์, ลอยกระทง). Templates reduce the blank-slate problem.
- **Soft / non-punishing planning**: don't turn lists into a "scoreboard the user is always losing against" (a common reason people quit to-do apps). Streaks with a grace period, positive framing ("ทำได้ X" not "เหลือ X").
- Payments via Thai channels (TrueMoney Wallet, QR) + Google Play Billing.

## Thai typography
- Recommended fonts: **Noto Sans Thai UI** (main body), **LINE Seed Sans TH** (friendly headlines/brand), **Sarabun** (formal/documents).
- **No spaces between Thai words** → must segment words for correct line-breaking/wrapping. Do NOT use `word-break: break-all` (splits mid-word). Use ICU BreakIterator (Android native) or PyThaiNLP segmentation.
- Avoid fonts lacking correct Thai line-breaking data; test every text-wrap point at multiple screen sizes.

## Thai NLP (for AI / smart-input features)
- Thai needs **word tokenization before understanding** (no word spaces).
- **PyThaiNLP** (Python, Apache-2.0) — tokenization, POS tagging, spell-check. https://pythainlp.org/
- Support Thai time/context words in a planner: พรุ่งนี้, มะรืนนี้, สัปดาห์หน้า, ปลายเดือน, ตอนเย็น, วันพระ, เงินเดือนออก, ทำบุญ.

## Research the Thai app market (Play Store review mining)
Find the exact feature/quality gap cheaply by reading **real Thai reviews** on the Play Store:
```
web_extract("https://play.google.com/store/apps/details?id=<pkg>&hl=th")
```
Returns downloads (e.g. "10M+"), rating, review count, and Thai-language reviews. Thai users openly post the gap — e.g. Todoist/Palu reviewers literally write "ไม่มีภาษาไทยเลย ถอนการติดตั้งครับ" and "อยากให้เพิ่มภาษาไทยเข้ามาด้วยค่ะ", which is direct, citable proof of localization demand. Also check App Store via `apps.apple.com/th/...`. This beats guessing and grounds the positioning in user quotes.

## Pitfalls
- Don't translate English UI copy literally into Thai — the register and politeness need to be re-derived from the target context.
- Don't assume Thai support = font exists. Rendering/line-breaking must be validated.
- **Dev-server port collision:** the user's Godforge dev server owns `:5173`. A new `vite` grabs the next free port — always confirm the *actual* bound port (read the server's own output), and run `npm run dev -- --port <unique> --strictPort` as its own `background=true` call (never chain `& sleep; curl` in a Hermes background command — it syntax-errors and the server never starts, so a curl of the assumed port silently hits Godforge instead). Details in the build reference.

## Supporting files
- `references/offline-first-app-build.md` — React+Vite+TS+Dexie→Capacitor local-first build, planner data model, rule-based Thai planner parser, dev-server port-collision fix.

## Supporting files
- `references/thai-language-rules.md` — full tables: คำทับศัพท์ errors, register→UI mapping, fonts, NLP libs, Thai vocab, source links.
