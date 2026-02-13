# Korean Beginner App — Development Guide

## Project Overview
Korean learning tool for beginners (TOPIK 1 level). Single-file web app deployed via GitHub Pages.

## Architecture
- `docs/index.html` — Single-file web app (HTML/CSS/JS, no build step)
- `docs/data/*.json` — Data served by GitHub Pages
- `docs/sw.js` — Service worker for offline support
- SM-2 spaced repetition algorithm (same as Anki)
- Firebase anonymous auth + Firestore for cross-device sync
- Light theme by default with dark mode toggle

## Data Files

| File | Description | Count |
|---|---|---|
| `docs/data/vocab.json` | Vocabulary entries by category | 1,898 entries, 33 categories |
| `docs/data/grammar.json` | Grammar patterns by category | ~50 patterns |
| `docs/data/error_drills.json` | Error correction drills | 80 drills |
| `docs/data/reading_drills.json` | Reading comprehension passages | 30 passages |
| `docs/data/dialogue_drills.json` | Dialogue practice drills | 25 dialogues |

### Vocab Entry Format
```json
{"korean": "...", "english": "...", "example": "...", "example_en": "...", "notes": "..."}
```

### Error Drill Format
```json
{"incorrect": "...", "correct": "...", "error_type": "...", "explanation": "..."}
```

### Reading Drill Format
```json
{"passage": "...", "passage_en": "...", "questions": [{"question": "...", "options": [...], "correct": 0, "explanation": "..."}]}
```

### Dialogue Drill Format
```json
{"title": "...", "context": "...", "turns": [{"speaker": "...", "line": "...", "options": [...]}], "explanation": "..."}
```

## Web App Features (Menu Items)
1. **Vocabulary Drill** — Flashcard review with SRS, Korean↔English
2. **Multiple Choice Quiz** — Auto-graded 4-option quiz
3. **Grammar Practice** — Pattern review + fill-in-the-blank
4. **Listening Drill** — TTS-based listening comprehension
5. **Mixed Review** — Weakest cards first across all categories
6. **Cloze Drill** — Fill-in-the-gap vocab in context
7. **Error Correction** — Find and fix grammar mistakes
8. **Reading Comprehension** — Korean passages with multiple-choice questions
9. **Dialogue Practice** — Multi-turn conversation practice with chat bubbles
0. **View Progress** — SRS stats, accuracy, streak

## Key Design Decisions
- Light theme with dark mode toggle (localStorage `beginner_korean_dark`)
- Self-rated flashcards (Anki-style) with "Show Answer" button for mobile
- Timed mode toggle (localStorage `beginner_korean_timed`)
- Streak tracking (localStorage `beginner_korean_streak`)
- SRS data (localStorage `beginner_korean_srs`)
- All drills are SRS-tracked with card IDs: `vocab:cat:idx`, `grammar:cat:idx`, `err:idx`, `read:idx`, `dlg:idx`
- UTF-8 encoding required for Korean text

## SRS Constants
```javascript
const AGAIN = 0, HARD = 2, GOOD = 3, EASY = 5;
```

## DrillEngine Architecture
All drills use a shared `DrillEngine()` that owns the lifecycle: session iteration, showCard, reveal, setupRating, rate, advance, showSummary. Each drill is a config object with:
- `renderCard(item, prog, onReveal)` — renders the card UI, calls `onReveal(result)` when ready
- `renderReveal(item, result, prog)` — renders the reveal UI with rating buttons. Return `false` to skip automatic `setupRating` (used by grammar exercises, reading, dialogue)
- `ratingDescs` — optional `{again, hard, good, easy}` labels for rating buttons

### Drill patterns
- **Standard drills** (Vocab, Grammar, Error Correction): `renderCard` → user interacts → `onReveal()` → `renderReveal` shows answer + rating buttons → DrillEngine wires up rating
- **Complex drills** (Reading, Dialogue): `renderCard` manages sub-steps (questions/turns) internally, calls `engine.setupRating(id)` directly on the last step. `renderReveal` returns `false`
- **Auto-graded option drills** (Multiple Choice, Listening, Cloze): Handle grading inline via `engine.autoGrade(id, isCorrect)` + `engine.autoAdvance()`

### Helper functions
- `drillHeader(title, prog)` — quit button + counter + progress bar HTML
- `ratingButtonsHtml(cardId, descs)` — rating buttons with interval predictions
- `getDistractors(entry, cat, field)` — picks 3 wrong options from vocab pool (module-level)

## Development Notes
- Large file: use incremental Edit, never Write the whole index.html
- Service worker cache version must be bumped on data file changes
- Firebase config is in index.html (do not commit real credentials)
- All data fetches have fallback paths (`data/` and `../data/`)
- `normalizeKorean()` strips whitespace/punctuation for answer comparison

## Deployment
- GitHub Pages: Settings → Pages → Source: Deploy from branch, Branch: main, Folder: /docs
- After any data change: bump service worker cache version in `docs/sw.js`
