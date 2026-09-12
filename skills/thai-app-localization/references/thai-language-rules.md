# Thai Language & Localization Reference (condensed)

Session-compiled knowledge bank for building Thai-language apps. Companion to SKILL.md.

## คำทับศัพท์ common errors (UI copy — use correct form)
| ถูกต้อง | เขียนผิดบ่อย |
|--------|------------|
| แอปพลิเคชัน | แอพพลิเคชั่น, แอพพลิเคชั่น |
| เบราว์เซอร์ | บราวเซอร์, เบราเซอร์ |
| คลิก | คลิ๊ก |
| อินเทอร์เน็ต | อินเตอร์เน็ต, อินเทอร์เนต |
| ดิจิทัล | ดิจิตอล, ดิจิทอล |
| ไฟล์ | ฟายล์ |

Rule: long-naturalized words keep dictionary form (ช็อกโกแลต, ก๊าซ, แก๊ส). Lookup: https://transliteration.orst.go.th/

## 5 register levels → UI mapping
| Level | Use for |
|-------|---------|
| พิธีการ | royal vocab; avoid in consumer UI |
| ทางการ | Terms, privacy, financial confirmations |
| กึ่งทางการ | feature descriptions, onboarding |
| ไม่เป็นทางการ/สนทนา | main UI buttons/status (default) |
| กันเอง | short push notifications |

Politeness particles ครับ/ค่ะ; soft tone beats imperative (เกรงใจ).

## Thai fonts
| Font | Use |
|------|-----|
| Noto Sans Thai UI | main body (safe layout) |
| LINE Seed Sans TH | friendly headlines/brand |
| Sarabun | formal/documents |
| Leelawadee UI / Tahoma | Windows dev/test offline (no web-font CDN needed) |

## Thai NLP libs
- **PyThaiNLP** (Python, Apache-2.0) — tokenize, POS, spell-check. https://pythainlp.org/
- ICU BreakIterator — on-platform Thai word breaking (Android built-in).

## Thai temporal/context vocab for planner AI
พรุ่งนี้, มะรืนนี้, สัปดาห์หน้า, ปลายเดือน, ตอนเย็น, วันพระ, เงินเดือนออก, ทำบุญ.

## Source references
- Royal Institute: https://royalsociety.go.th/
- Transliteration lookup: https://transliteration.orst.go.th/
- Noto Sans Thai: https://fonts.google.com/noto/specimen/Noto+Sans+Thai
- LINE Seed Sans TH: https://seed.line.me/index_th.html
- PyThaiNLP: https://pythainlp.org/
- Localization research reports: `C:\Users\Acer\Desktop\Planner-App\research\` (thai-planner-app-research.md, planner-app-ux-ui.md, build-technical.md)
