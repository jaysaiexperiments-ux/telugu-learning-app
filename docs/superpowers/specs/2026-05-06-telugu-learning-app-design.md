# Telugu Learning App — Design Spec
**Date:** 2026-05-06
**Repo:** `jaysaiexperiments-ux/telugu-learning-app`
**Deployment:** GitHub Pages (main branch, root `/`)

---

## Goal

A fully client-side Telugu language learning web app for a native English speaker
starting from zero. Science-backed methods (SRS, comprehensible input, sentence
mining, active recall, spaced repetition, interleaving) implemented in a static
site deployable to GitHub Pages with no backend.

---

## File Structure

```
telugu-learning-app/
├── index.html       # App shell + all JS logic + inline CSS
├── words.js         # 200-word corpus (loaded via <script src>)
├── sw.js            # Service Worker for daily push notifications
└── README.md        # GitHub Pages deployment steps
```

---

## Tech Stack

- Vanilla JS + HTML + CSS (no frameworks)
- localStorage for all persistence
- Anthropic API (`claude-sonnet-4-20250514`) via browser fetch for chat
- Service Worker for local daily notifications
- Google Fonts: Noto Sans Telugu
- GitHub Pages for deployment

---

## Data Model

### Word Object (`words.js`)
```js
{
  id: Number,
  english: String,
  telugu: String,                        // Telugu script
  transliteration: String,               // Latin phonetic
  pronunciation_hint: String,            // e.g. "nee-roo"
  example_sentence_telugu: String,
  example_sentence_english: String,
  semantic_group: String,                // greetings | numbers | family | food |
                                         // body | time | directions | emotions |
                                         // daily_life | verbs_basic | adjectives_basic
  frequency_rank: Number                 // 1 = most frequent
}
```

### SRS Card State (localStorage `srs_cards`)
```js
{
  [wordId]: {
    interval: Number,       // days until next review
    easeFactor: Number,     // SM-2 ease factor, starts at 2.5
    repetitions: Number,    // successful reviews in a row
    nextReview: String,     // ISO date string
    lastRating: Number      // 0-5
  }
}
```

### User Progress (localStorage `user_progress`)
```js
{
  level: "beginner" | "elementary" | "intermediate",
  xp: Number,
  streak: Number,
  last_visit: String,          // ISO date
  notification_time: String,   // "HH:MM", default "08:00"
  session_stats: [             // last 7 days
    { date: String, accuracy: Number, cards_reviewed: Number }
  ]
}
```

### Other localStorage Keys
- `placement_done` — boolean
- `chat_history` — Message[] (role + content pairs for Anthropic API)
- `api_key` — user's Anthropic API key

---

## SRS Engine (SM-2 Algorithm)

After each card rating (0–5):
```
if rating < 3:
  repetitions = 0
  interval = 1
else:
  if repetitions == 0: interval = 1
  elif repetitions == 1: interval = 3
  else: interval = Math.round(interval * easeFactor)
  repetitions += 1

easeFactor = easeFactor + (0.1 - (5 - rating) * (0.08 + (5 - rating) * 0.02))
easeFactor = Math.max(1.3, easeFactor)
nextReview = today + interval days
```

**Session budget:**
- New cards per session: `Math.max(5, 20 - reviewsDue)` (capped at 20 total cards)
- Reviews always take priority over new cards
- As deck grows, reviews dominate — new card rate naturally slows

---

## Auto-Pacing Engine

**Placement quiz (Day 1, 10 questions):**
- Pattern recognition questions (match Telugu letter → sound)
- No prior Telugu knowledge assumed
- Score 0–3 → Beginner | 4–6 → Elementary | 7–10 → Intermediate

**Difficulty adjustment after each session:**
- Accuracy > 85% → increase new cards by 2 next session
- Accuracy < 60% → reduce new cards by 2, flag for review

**Level targets:**
- Week 1: Telugu script basics + 50 core words
- Month 1: 300 words + basic sentences
- Month 3: 700 words + conversational phrases

---

## UI Architecture

### Layout
- Mobile-first, dark mode by default
- Color theme: `#0a1628` bg, `#c9a84c` gold accent, `#1e3a5f` card surface
- Bottom tab bar: 📚 Learn | 💬 Chat | 📊 Progress
- Font: Noto Sans Telugu (Google Fonts CDN)

### Tab: Learn
- SRS flashcard session (current session cards)
- Card flip: CSS 3D transform, English front → Telugu back
- Rating buttons (1–5) appear after flip
- Module selector at top (auto-unlocked by level)
- Progress bar for today's session
- Streak + XP badge in header

### Tab: Chat (WhatsApp-style)
- Message bubbles (user right, tutor left)
- Three mode pills: 📖 Explain | 🧠 Quiz me | 💬 Free chat
- API key modal (gear icon, top-right) — key stored in localStorage
- Typing indicator while API call in flight
- Each message sends full `chat_history` to maintain context

### Tab: Progress
- Streak counter (flame icon)
- XP bar with level label
- Total words learned / total in deck
- 7-day accuracy bar chart (Canvas API)
- Next 5 review times listed
- Module completion grid

### First-Run Flow
1. Notification permission prompt (browser API)
2. 10-question placement quiz
3. Level assigned → module gates set
4. Redirect to Learn tab, first session begins

### Module Unlock Gates
- Beginner: Modules 1–2
- Elementary: Modules 1–3
- Intermediate: All 6 modules

---

## Content Modules

| Module | Topic | Words | Unlocked at |
|--------|-------|-------|-------------|
| 1 | Phonetics & Script Basics (vowels + consonants) | — | Beginner |
| 2 | Survival Telugu (greetings, yes/no, numbers 1–20) | 30 | Beginner |
| 3 | Daily Life (food, family, time, directions) | 50 | Elementary |
| 4 | Sentence Patterns (SOV, verb conjugation, tense) | 40 | Elementary |
| 5 | Conversations (shopping, transport, restaurants) | 50 | Intermediate |
| 6 | Intermediate Grammar (postpositions, conditionals) | 30 | Intermediate |

Each module contains: flashcard deck + mini lesson text + sample dialogue + end quiz.

---

## AI Chat

**Endpoint:** `https://api.anthropic.com/v1/messages`
**Model:** `claude-sonnet-4-6`
**Auth:** User's own API key (pasted in settings modal, stored in localStorage)

**Required headers:**
```
x-api-key: <user_key>
anthropic-version: 2023-06-01
content-type: application/json
anthropic-dangerous-direct-browser-access: true
```

**System prompt (enforced persona):**
> You are Priya, a friendly Telugu language tutor for native English speakers. Always respond in English first, then Telugu script, then transliteration in parentheses. Keep explanations short and chunked. Always give at least one example sentence. Correct the user's Telugu gently — show the correct form, explain why briefly. Never use jargon. If the user asks "how do I say X?", answer immediately then ask them to try using it in a sentence.

**Session memory:** Full `chat_history` array passed on every call.

---

## Service Worker & Notifications

- `sw.js` registered on first page load
- On install: schedules a daily local notification at `user_progress.notification_time`
- Notification fires via `self.registration.showNotification()`
- Since true background push requires a server, SW reschedules on each page open
- In-app banner shown if `last_visit` date ≠ today (regardless of SW)

---

## Semantic Groups in `words.js`

Cover all 11 groups across 200 words:
`greetings` | `numbers` | `family` | `food` | `body` | `time` | `directions` | `emotions` | `daily_life` | `verbs_basic` | `adjectives_basic`

Words ordered by `frequency_rank` (1 = most common spoken Telugu).

---

## GitHub Pages Deployment

1. Create repo `jaysaiexperiments-ux/telugu-learning-app` (public)
2. Push `main` branch with all files at root
3. GitHub Settings → Pages → Source: main branch, / (root)
4. App live at `https://jaysaiexperiments-ux.github.io/telugu-learning-app/`

---

## Out of Scope

- Backend / server of any kind
- User accounts / cloud sync
- Audio pronunciation (future enhancement)
- Native mobile app
- Multiple user profiles
