# Talmeez product rules (from the MVP prototype v20, Sep 2026)

Prototype: https://claude.ai/artifact/GEqttKuprtUB9irMoNURQu

**Brand:** gradient T mark + "Talmeez" as live text (Inter 600, blue gradient #2A5AD4 → #4B7BDC). No ".me".
**Trust badge:** "Identity verified" (Identité vérifiée / هوية موثّقة). Never "verified tutor".

## Accounts and verification
- Email verified with a 6-digit code (Supabase Auth via Resend). No SMS.
- Phone collected and format-checked for parents and tutors, never texted. Any valid international number is accepted and stored as E.164; typed without `+` it is read as Lebanese. The WhatsApp hand-off (`wa.me/<digits>`) works with any country code.
- **Identity verified** = an admin approved the identity document AND the photo has not changed since without an admin looking at it again. When an approved tutor replaces or removes their photo, the badge pauses (profile, search, Ask Talmeez and MCP all read `tutors_public.id_verified`; the profile stays live) and the change appears on the admin Verification tab as "Photo changed, compare with the approved ID" (old photo, new photo, the approved ID through the logged signed link). Approve restores the badge; reject keeps it off and emails the tutor. Edit my profile warns before uploading: "Changing your photo pauses your Identity verified badge until we check it".
- Tutors never see a parent's full name, email or phone — only "Nadine M." — unless the parent shares a number in chat.

## Admin access
- Admins can consult everything: users directory (full name, email, phone, join date, how they heard, child's program/class/also-follows/area, tutor details incl. whether an ID is on file), any conversation with real names, reports (no written reason needed), identity documents (kept after approve/reject).
- Every profile view, conversation opening, report opening and document view goes to `admin_access_log`.
- The privacy policy must say admins can read conversations and keep ID documents (some users are minors).

## School programs and classes
- **Programs:** Lebanese · French · American · IB · British · Exams & tests · University (+ "Other" free text in sign-up).
- **Classes a child can be in** — full list with codes and stages in `packages/catalog/catalog.json`.
- **What tutors pick** — ranges per program, also in the catalog.
- **Equivalents:** every class has a stage (0 = KG, 1–12 = Grade 1–12, 13 = university). Same stage = same class across programs (EB9 = 3e = Grade 9 = British Year 10). Class buttons show equivalents ("= 3e · Grade 9").
- **Matching:**
  - Exact: tutor teaches that class in the same program (Lebanese S3 tracks must match: SG ≠ LH).
  - Near: tutor teaches the same stage in another school program. Shown after exact matches under "Same class in other programs", amber chip "Same class, French program".
  - Tests and university: exact only.
  - Nobody teaches the exact class → "No tutor lists {class} yet." then near matches.
- Still to confirm with 2 teachers: Lebanese S1–S3 naming and new-curriculum names.

## Flows
- **Parent browse:** search (subject · class via program picker · area). Results quick bar always visible (price chips Any/$10/$15/$20; sort). Full filters (subject, class, area, max price $5–$50, "include online"). Sort: Recommended, Price, Highest rated. Tutors are located by city / area (not neighbourhood), so there is no distance filter and no "Nearest" sort.
- **Ask Talmeez chat:** AR/FR/EN (+ Arabizi), up to 3 matches each with a one-line reason; says when a match is another program; keyword fallback.
- **Tutor profile:** details as label/value list; classes grouped by program; Background (situation, studies, experience, certificates) marked "Stated by the tutor. Talmeez verifies identity, not diplomas."
- **Parent sign-up (3 steps):** 1) full name, email + code, phone · 2) child: program (tiles, or Other) → class (buttons with equivalents) → "Also follows (optional)" → place (autocomplete: type, pick a town, district or region) → optional subjects; child's name never asked · 3) how did you hear about Talmeez.
- **Tutor sign-up (6 steps):** 1) required photo ("You have more chances to get booked with a clear photo of yourself…"), full name, email + code, phone (never public), where they teach, then (only for home / tutor-place lessons) the places they cover, typed into an autocomplete and picked from suggestions as removable chips (towns, districts, governorates or places abroad; several allowed; online-only tutors give none; places only abroad count as online), languages · 2) studies & experience (situation, highest studies, field, university: LU, AUB, LAU, USJ, NDU, USEK, Balamand, BAU, AUST, LIU, Haigazian, Other; experience; certificates) · 3) subjects → programs taught → per program a block of its ranges → rate; unticking a program removes its classes · 4) bio ≥ 15 chars with counter + AI rewrite · 5) how did you hear · 6) identity document ("Any Lebanese identification, or a passport or international ID. Only Talmeez admins can see it.").
- **Messaging:** lead = 2 parent messages + 1 tutor reply, once per pair. Handoff tracking: WhatsApp button + phone detection. Booking prompt after 48h (or right away on a booking signal). Handoffs never blocked.
- **Admin tabs:** Verification (photo next to document) · Users · Conversations · Reports (reported messages, reviews waiting, published reviews) · Insights · Settings.

## Translation of user content
- Reviews, tutor replies and bios shown in the visitor's language with "Auto-translated from French" + "Show original" toggle (cards: silent; home testimonials: note, no toggle).
- On save, an Edge Function translates into the 2 other languages → `content_translations` (source table, id, field, lang, text, source hash). Edits or admin redactions discard and redo translations. Admins see the original. Translations never add or soften; redacted names stay redacted.

## "Other" options
Every sign-up choice list has "Other" with free text: program (+ class), subjects, programs taught, university, how they heard. **Exception: places are never free text** (autocomplete, you must pick a suggestion; one stable code per place, label shown in the user's language: "Jounieh · Keserwan, Mount Lebanon", "Metn · Mount Lebanon", "Mount Lebanon", "Barcelona · Spain"), so "Beyrouth" and "Beirut" can never become two places. Search matches by hierarchy (a tutor in Jounieh is found by Jounieh, Keserwan or Mount Lebanon). The list is `docs/places.md`. Subjects include SAT and IELTS.

## Reviews
1. Eligibility: lead conversation + contact exchange + "Yes, I booked".
2. Tutor confirmation → labels "Booking confirmed by both" / "Booking confirmed by parent". No tutor answer in 7 days doesn't block. Tutor says no → held or reported.
3. Admin keeps or hides (logged).
4. One review per parent per tutor, from a verified email.
5. Burst flag: 3+ reviews for one tutor within hours from new accounts are held.
6. Wording: "Booking confirmed", never "verified lesson".
- Admins can't rewrite reviews or change ratings. They can remove personal details (shows "Edited by Talmeez to remove personal details", previous version kept) and ask the parent to edit. Parents can edit within 30 days ("Edited").

## Paywall
- Tutor subscription price and free leads (2) read from `settings`; changing `free_leads` in admin changes behaviour at once, no deploy.
- **A free lead is used only by a real lead**: a counted lead (the parent sent at least 2 messages AND the tutor replied, `conversations.lead_counted_at`). Unanswered and one-message conversations never use one, so fake parents who get no answer cannot burn a tutor's free leads.
- While counted leads < `free_leads`, every new conversation is open: the tutor reads it and can reply. Once counted leads reach `free_leads`, NEW conversations (not a counted lead, tutor never replied in them) wait behind the paywall: the parent is unaffected, the tutor sees "N parents are waiting", the `lead_notice` email is still sent without text. Counted leads and chats the tutor already answered stay open, whatever happens to the plan or to `free_leads` later.
- The dashboard lead meter and the plan page show the same number (counted leads of `free_leads`). Several open conversations answered at the same moment can end a lead or two over `free_leads`; nothing is taken away, the next new conversation waits.
- Admin activates plans by hand at launch (Whish or bank transfer); activating or extending a plan opens everything that waits, for good.
