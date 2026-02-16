# Watcher: Movie Swipe App

## Project Overview

**Watcher** is a shared movie watchlist app that helps couples and friend groups decide what to watch together. The core mechanic is a "Tinder for movies" swiping interface where users independently swipe on trailers from a shared list, and matches surface when preferences align.

---

## Jobs-to-be-Done

**Primary JTBD:** "When my partner and I sit down to watch something, help us quickly converge on a movie we'll both enjoy—without the endless 'I don't know, what do you want to watch?' loop."

**Secondary JTBD:** "Give me a place to save movie recommendations throughout the week so I don't forget them when it's actually time to watch."

---

## Target Users

### Persona 1: "Date Night Duo"
- **Who:** Couples (primary target)
- **Behavior:** Save movie ideas throughout the week from social media, Netflix thumbnails, friend recommendations
- **Pain point:** Saturday night arrives, they've forgotten what looked good, end up scrolling Netflix for 30 minutes
- **Desired outcome:** Quick, gamified decision-making that feels fun rather than frustrating

### Persona 2: "Watch Party Friends"
- **Who:** Groups of 3-5 friends
- **Behavior:** Each person has different taste; suggestions get lost in group chats
- **Pain point:** Democratic decision-making is slow; someone always compromises
- **Desired outcome:** Voting mechanism that surfaces consensus without awkward negotiation

---

## Current State (POC v1)

### What's Built
A single-page React app (served as static HTML) with:
- **Setup screen:** JSON textarea to input movie list
- **Swipe screen:** Embedded YouTube trailer with Yes/No/Watch buttons
- **Results screen:** Shows "shortlist" of movies marked Yes

### Tech Stack
- React 18 via CDN (no build step)
- Tailwind CSS via CDN
- Flask serving static files
- No database (state resets on refresh)

### Data Format
```json
[
  { "title": "Movie Name", "youtube": "VIDEO_ID_OR_URL" },
  { "title": "Another Movie", "youtube": "dQw4w9WgXcQ" }
]
```

### File Structure
```
/watcher
├── app.py              # Flask server
└── static/
    └── index.html      # Self-contained React app
```

### Running Locally
```bash
cd /watcher
python app.py
# Open http://localhost:5000
```

---

## Roadmap

### Phase 1: Local Persistence (Next)
**Goal:** Movie list survives page refresh

**Tasks:**
- [ ] Store movie list in localStorage
- [ ] Add "Add Movie" form (title + YouTube URL)
- [ ] Add "Remove Movie" button per item
- [ ] Add "Edit List" mode to manage saved movies

**Acceptance criteria:** User can build a list over multiple sessions without re-entering data.

---

### Phase 2: TMDB Integration
**Goal:** Rich movie metadata without manual entry

**Tasks:**
- [ ] Integrate TMDB API for movie search
- [ ] Auto-fetch: poster, synopsis, runtime, genres, release year
- [ ] Auto-fetch YouTube trailer via TMDB's video endpoint
- [ ] Display poster cards instead of just titles in list view
- [ ] Add "Search TMDB" input as alternative to manual YouTube URL

**API Reference:**
- TMDB API docs: https://developer.themoviedb.org/docs
- Search endpoint: `GET /search/movie?query={title}`
- Videos endpoint: `GET /movie/{id}/videos` (returns YouTube trailer keys)
- Image base URL: `https://image.tmdb.org/t/p/w500{poster_path}`

**Data model evolution:**
```json
{
  "tmdb_id": 693134,
  "title": "Dune: Part Two",
  "poster": "/path.jpg",
  "youtube": "Way9Dexny3w",
  "year": 2024,
  "genres": ["Sci-Fi", "Adventure"],
  "runtime": 166,
  "added_by": "user_id",
  "added_at": "2024-01-15T20:00:00Z"
}
```

---

### Phase 3: Multi-Device Sync (Core Feature)
**Goal:** Two people swipe on their own phones, matches detected in real-time

**Architecture options:**

#### Option A: WebSocket Server (Recommended for MVP)
- Flask-SocketIO or FastAPI with WebSockets
- Room-based sessions (join via code or QR)
- Server broadcasts swipe events, detects matches

#### Option B: Firebase Realtime Database
- No backend to manage
- Realtime listeners for swipe state
- Easier for prototype, harder to extend

**Session flow:**
1. Host creates session → gets 4-digit code + QR
2. Guest joins via code or QR scan
3. Both see same movie queue (shuffled identically via seeded RNG)
4. Each swipe is sent to server
5. Server detects match (both swiped Yes on same movie)
6. Match triggers celebration UI on both devices

**Data model:**
```json
{
  "session_id": "ABCD",
  "host_id": "uuid",
  "guest_id": "uuid",
  "movie_queue": ["tmdb_id_1", "tmdb_id_2", ...],
  "swipes": {
    "host": { "693134": "yes", "872585": "no" },
    "guest": { "693134": "yes" }
  },
  "matches": ["693134"],
  "created_at": "...",
  "expires_at": "..."
}
```

---

### Phase 4: Streaming Availability
**Goal:** Show where to watch and deep-link to streaming apps

**Tasks:**
- [ ] Integrate JustWatch API or TMDB's watch/providers endpoint
- [ ] Display streaming icons (Netflix, Prime, etc.) per movie
- [ ] "Watch Now" button deep-links to streaming app if available
- [ ] Filter movies by "available on services I have"

**TMDB watch providers endpoint:**
`GET /movie/{id}/watch/providers` → returns by region

---

### Phase 5: User Accounts & Shared Lists
**Goal:** Persistent identity, async list building

**Tasks:**
- [ ] Auth (email/password or OAuth)
- [ ] Create/join shared lists (not just sessions)
- [ ] "Add to list" bookmarklet or share extension
- [ ] Push notification when partner adds a movie
- [ ] Watch history with ratings

---

### Phase 6: Smart Recommendations
**Goal:** Help users discover movies they'd both like

**Tasks:**
- [ ] Track match patterns (genres, directors, actors)
- [ ] Suggest movies based on mutual taste profile
- [ ] "Surprise Us" mode: app picks from likely matches

---

## Technical Decisions

### Why no build step?
For rapid prototyping, CDN-loaded React eliminates npm/webpack complexity. Migrate to Vite or Next.js when:
- Bundle size matters (production)
- You need SSR or routing
- Component library grows beyond single file

### Why Flask?
Simple, familiar, sufficient for static serving + future API routes. Swap for FastAPI if you need async WebSockets natively.

### Why TMDB over OMDb?
- Free tier is generous (1000 requests/day)
- Includes trailers, posters, streaming availability
- Better international coverage

---

## UI/UX Principles

1. **Speed over completeness:** The "what do we watch" moment is impatient. Every tap should feel instant.
2. **Gamification:** Swiping should feel like a game, not a chore. Celebrate matches loudly.
3. **Low-friction onboarding:** First session should work in <30 seconds. No signup required for MVP.
4. **Mobile-first:** Primary use case is two people on a couch with phones.

---

## Open Questions

- **Tie-breaker:** If no matches after full queue, what happens? Random pick from "maybes"? Extend queue?
- **Asymmetric lists:** Can users have different queues, or must they be identical?
- **Trailer autoplay:** Autoplay is engaging but burns data. Make it optional?
- **Offline support:** Worth caching trailers/posters for spotty wifi?

---

## Quick Commands

```bash
# Run locally
cd /watcher && python app.py

# Test with sample data
# Paste this into the JSON textarea:
[
  {"title": "Dune: Part Two", "youtube": "Way9Dexny3w"},
  {"title": "Oppenheimer", "youtube": "uYPbbksJxIg"},
  {"title": "Poor Things", "youtube": "RlbR5N6veqw"},
  {"title": "The Holdovers", "youtube": "AhKLpJmHhIg"},
  {"title": "Past Lives", "youtube": "kA244xewjcI"}
]
```

---

## Prompt for Generating Movie Lists

Use this with any Claude instance to get test data:

> Generate a JSON array of 15 popular movies from 2023-2025 with their official YouTube trailer video IDs. Each object needs `title` (string) and `youtube` (string, just the 11-character video ID). Output only valid JSON, no markdown fences or explanation.

---

## Contact / Context

This project was started as a POC to validate the "swipe to match" mechanic for couples deciding what to watch. The north star is reducing decision friction while making the process feel collaborative rather than compromising.
