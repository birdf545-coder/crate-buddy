# 🎧 Crate Buddy

> The perfect AI assistant for DJs and music collectors.

Crate Buddy is a Telegram-based AI assistant that gives DJs a 10x productivity boost by automating music organization, research, and purchasing — all via voice commands.

---

## The Problem

DJs spend hours every week on:
1. **Researching** new music across YouTube, SoundCloud, Bandcamp, Discogs, etc.
2. **Buying** tracks across multiple stores
3. **Organizing** playlists in Rekordbox before every gig

Crate Buddy solves all three.

---

## Core Features

### 🗂️ Playlist Organizer
- Analyzes your full music library
- Learns your style and preferences
- Creates playlists by voice command: *"Make me a Techno 130bpm playlist from these artists"*
- Exports Rekordbox-compatible XML files

### 🔍 Music Researcher
- Searches by genre, era, BPM, mood, or artist
- Monitors releases from your favorite artists
- Discovers related artists based on your taste
- Tracks gigs and events from artists you follow
- Sources: Spotify, SoundCloud, YouTube, Bandcamp, Discogs, Beatport + more

### 🛒 Purchase Assistant
- Finds buy links across all major stores
- **Affiliate revenue:** Beatport (15%), Bandcamp, Juno + others
- Sends curated purchase recommendations with direct links

---

## Architecture

```
User (Telegram voice/text)
        │
        ▼
┌───────────────────┐
│   Crate Buddy     │  ← Main orchestrator agent
│   (Telegram Bot)  │
└───────┬───────────┘
        │
   ┌────┴─────┬──────────────┐
   ▼          ▼              ▼
┌──────┐  ┌────────┐  ┌──────────┐
│ Lib  │  │Research│  │ Purchase │
│Agent │  │ Agent  │  │  Agent   │
└──────┘  └────────┘  └──────────┘
   │          │              │
   ▼          ▼              ▼
Rekordbox  Spotify/SC/    Beatport/
  XML      YouTube/       Bandcamp/
  export   Discogs API    Juno links
```

### Subagents

| Agent | Role |
|-------|------|
| **Library Agent** | Parses music library, manages playlists, exports Rekordbox XML |
| **Research Agent** | Searches music platforms via API, monitors artists, finds new releases |
| **Purchase Agent** | Finds buy links, injects affiliate codes, sends recommendations |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Interface | Telegram Bot (voice + text) |
| Orchestrator | Python / OpenClaw agents |
| Library parsing | Rekordbox XML parser |
| Music APIs | Spotify API, Discogs API, SoundCloud API, YouTube Data API |
| Purchase stores | Beatport, Bandcamp, Juno, Traxsource |
| Storage | Supabase (user libraries, preferences, history) |
| Deployment | Vercel / Railway |

---

## Monetization

- **Subscription:** Monthly/yearly per DJ
- **Affiliate commissions:** 
  - Beatport: 15% per sale
  - Bandcamp, Juno, Traxsource: % per sale
  - All purchase links routed through affiliate program

---

## Roadmap

- [ ] MVP: Library parser + Rekordbox XML export via Telegram command
- [ ] v0.2: Playlist creation by genre/BPM/artist
- [ ] v0.3: Research agent (Discogs + Beatport API)
- [ ] v0.4: Purchase agent with affiliate links
- [ ] v0.5: Spotify + SoundCloud integration
- [ ] v1.0: Full voice-command workflow + dashboard

---

## Team

- **J** — Product vision, DJ domain expertise
- **Valentín** — Development
- **Marko** — Research & integrations
- **Silvana** — Development
- **Gaia** — AI architecture & automation

---

## Contributing

Branches per feature — no direct commits to `main`.

```bash
git checkout -b feature/your-feature-name
# work
git push origin feature/your-feature-name
# open PR
```
