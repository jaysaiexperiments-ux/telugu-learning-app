# తెలుగు — Telugu Learning App

A science-backed Telugu language learning app using spaced repetition (SM-2), AI chat tutor, and 200 core vocabulary words. Fully client-side — works offline after first load.

## Deploy to GitHub Pages

1. **Create the repo** (already done if you're reading this)

2. **Push the code**
   ```bash
   git remote add origin https://github.com/jaysaiexperiments-ux/telugu-learning-app.git
   git push -u origin main
   ```

3. **Enable GitHub Pages**
   - Go to your repo on GitHub
   - Settings → Pages
   - Source: **Deploy from a branch**
   - Branch: `main`, folder: `/ (root)`
   - Click Save

4. **Your app is live at:**
   ```
   https://jaysaiexperiments-ux.github.io/telugu-learning-app/
   ```
   (takes ~1 minute to go live)

## Using the App

### First Run
- Answer the 10-question placement quiz to set your starting level
- Your level unlocks modules progressively

### Learn Tab (📚)
- Flashcards use the **SM-2 spaced repetition** algorithm (same as Anki)
- See the English word → recall the Telugu → flip to check → rate 1–5
- Rating 1 (Again) = review tomorrow · Rating 5 (Perfect) = review in weeks

### Chat Tab (💬)
- Requires your own **Anthropic API key** (get one at console.anthropic.com)
- Enter it via the ⚙️ gear icon — stored only in your browser
- Modes: Explain grammar · Quiz me · Free conversation · Sentence help

### Progress Tab (📊)
- Streak counter, XP, accuracy chart, upcoming review times
- Set your daily notification reminder time

## Adding Your Own Words

Open `words.js` and add an object to the `WORDS` array:

```js
{
  id: 201,                                    // unique id
  english: "cloud",
  telugu: "మేఘం",
  transliteration: "mēghaṁ",
  pronunciation_hint: "ME-ghum",
  example_sentence_telugu: "ఆకాశంలో మేఘాలు ఉన్నాయి",
  example_sentence_english: "There are clouds in the sky",
  semantic_group: "daily_life",               // any existing group
  frequency_rank: 201
}
```

Save and refresh — the word enters your SRS deck automatically.

## Science Behind the App

| Technique | Implementation |
|-----------|---------------|
| Spaced Repetition (SM-2) | Cards scheduled at 1→3→7→14→30 day intervals |
| Active Recall | English shown first; user produces Telugu before seeing answer |
| Comprehensible Input (i+1) | Modules unlock by level; new words always in example sentences |
| Pareto Principle | Top-200 frequency words cover ~80% of spoken Telugu |
| Sentence Mining | Every word shown in a real sentence, not in isolation |
| Interleaving | All semantic groups mixed in each session |
| Habit Stacking | Daily streak counter + XP + notification reminders |

## Tech Stack

- Vanilla JS + HTML5 + CSS3 — no frameworks, no build step
- `localStorage` for all progress data
- SM-2 algorithm implemented from scratch
- Anthropic `claude-sonnet-4-6` API via browser fetch
- Service Worker for offline caching
- Canvas API for progress charts
- Google Fonts: Cormorant Garamond + DM Sans + Noto Sans Telugu
