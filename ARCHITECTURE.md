# Architecture — Crate Buddy

## Overview

Crate Buddy is a multi-agent system orchestrated via Telegram. The user interacts exclusively through voice or text messages. The main bot delegates tasks to specialized subagents.

---

## System Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        USER                                  │
│              (Telegram voice / text)                         │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                 CRATE BUDDY BOT                              │
│              (Main Orchestrator)                             │
│                                                             │
│  • Receives voice → transcribes (Whisper)                   │
│  • Parses intent (NLU)                                       │
│  • Routes to correct subagent                               │
│  • Returns formatted response to user                       │
└──────┬──────────────┬──────────────────┬────────────────────┘
       │              │                  │
       ▼              ▼                  ▼
┌──────────┐  ┌──────────────┐  ┌──────────────┐
│ LIBRARY  │  │  RESEARCH    │  │  PURCHASE    │
│  AGENT   │  │   AGENT      │  │   AGENT      │
└──────────┘  └──────────────┘  └──────────────┘
```

---

## Agent Specs

### Main Orchestrator
**Responsibilities:**
- Voice-to-text transcription (Whisper)
- Intent classification: organize / research / purchase / mixed
- Session management per user
- Response formatting and delivery

**Input:** Telegram message (voice or text)  
**Output:** Telegram reply + optional file attachment

---

### Library Agent
**Responsibilities:**
- Parse Rekordbox XML library
- Index tracks: title, artist, BPM, key, genre, energy, date added
- Create/update playlists based on criteria
- Export Rekordbox-compatible XML
- Store library in Supabase for fast queries

**Key operations:**
```
parse_library(xml_file) → indexed tracks in DB
create_playlist(filters: {genre, bpm_range, artists, energy}) → playlist
export_rekordbox(playlist_id) → XML file
get_track_stats(user_id) → library overview
```

**Formats supported:**
- Rekordbox XML (primary)
- M3U (secondary)

---

### Research Agent
**Responsibilities:**
- Search for tracks/releases by genre, era, BPM, mood
- Monitor new releases from followed artists
- Discover related artists
- Find upcoming gigs and events
- Return ranked results with metadata

**APIs integrated:**
| Platform | API | Data |
|----------|-----|------|
| Beatport | REST API | Charts, releases, genres |
| Discogs | REST API | Catalog, releases, artists |
| Spotify | Web API | Artist data, related artists, audio features |
| SoundCloud | API | Tracks, playlists |
| YouTube | Data API v3 | DJ sets, mixes |
| Bandcamp | (scrape/fan API) | Independent releases |
| Resident Advisor | Scrape | Events, artist pages |

**Key operations:**
```
search_music(query: {genre, era, bpm, mood, artist}) → ranked tracks
get_new_releases(artists: []) → latest releases
find_related_artists(artist_name) → similar artists
get_artist_gigs(artist_name) → upcoming events
monitor_artists(user_id) → notification queue
```

---

### Purchase Agent
**Responsibilities:**
- Find purchase links for specific tracks
- Inject affiliate codes per store
- Compare prices across stores
- Queue purchase recommendations

**Affiliate programs:**
| Store | Commission | Affiliate Program |
|-------|-----------|-------------------|
| Beatport | 15% | Impact / direct |
| Juno Download | TBD | Direct |
| Traxsource | TBD | Direct |
| Bandcamp | TBD | Direct |

**Key operations:**
```
find_purchase_links(track: {title, artist}) → [{store, price, affiliate_url}]
build_recommendation(tracks: []) → formatted message with links
track_clicks(user_id, link) → affiliate attribution
```

---

## Data Models

### Track
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "title": "string",
  "artist": "string",
  "bpm": 130,
  "key": "11A",
  "genre": "Techno",
  "energy": 8,
  "duration_sec": 420,
  "date_added": "2026-01-15",
  "file_path": "string",
  "rekordbox_id": "string",
  "tags": ["dark", "industrial"]
}
```

### Playlist
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "name": "string",
  "filters_used": {},
  "track_ids": ["uuid"],
  "created_at": "timestamp",
  "exported_at": "timestamp"
}
```

### UserPreferences
```json
{
  "user_id": "uuid",
  "favorite_genres": ["Techno", "Industrial"],
  "bpm_range": {"min": 125, "max": 140},
  "followed_artists": ["string"],
  "purchase_stores": ["Beatport", "Juno"]
}
```

---

## Telegram Bot Flow

```
1. User sends voice note
2. Bot downloads audio
3. Whisper transcribes → text
4. NLU classifies intent
   ├── "organize" → Library Agent
   ├── "research" → Research Agent
   ├── "buy" → Purchase Agent
   └── "mixed" → orchestrate multiple agents
5. Agent executes task
6. Bot formats response
7. Bot sends reply (text + optional file)
```

---

## File Structure

```
crate-buddy/
├── README.md
├── ARCHITECTURE.md
├── .env.example
├── requirements.txt
│
├── bot/
│   ├── main.py              # Telegram bot entry point
│   ├── handlers.py          # Message handlers
│   ├── transcription.py     # Whisper voice-to-text
│   └── intent.py            # NLU intent classification
│
├── agents/
│   ├── orchestrator.py      # Main routing logic
│   ├── library_agent.py     # Library management
│   ├── research_agent.py    # Music discovery
│   └── purchase_agent.py    # Buy links + affiliate
│
├── parsers/
│   ├── rekordbox.py         # Rekordbox XML parser/exporter
│   └── m3u.py               # M3U parser
│
├── integrations/
│   ├── beatport.py
│   ├── discogs.py
│   ├── spotify.py
│   ├── soundcloud.py
│   └── youtube.py
│
├── db/
│   ├── supabase.py          # DB client
│   └── models.py            # Data models
│
└── utils/
    ├── affiliate.py         # Link injection
    └── formatter.py         # Response formatting
```

---

## Environment Variables

```env
TELEGRAM_BOT_TOKEN=
SUPABASE_URL=
SUPABASE_KEY=
SPOTIFY_CLIENT_ID=
SPOTIFY_CLIENT_SECRET=
DISCOGS_TOKEN=
BEATPORT_API_KEY=
YOUTUBE_API_KEY=
SOUNDCLOUD_CLIENT_ID=
OPENAI_API_KEY=          # for Whisper / NLU fallback
```
