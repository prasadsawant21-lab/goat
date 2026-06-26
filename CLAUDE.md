# CLAUDE.md — Football Platform (GoatScore / PitchIQ)
> Claude Code instruction file. Read this fully before touching any file.

---

## Project Overview

You are building a **real-time AI-powered football analytics platform** for the MENA market (English + Arabic). It started as a single `index.html` on GitHub Pages and is being upgraded to a full Next.js application with a live database, real sports data, and Claude AI commentary.

**Live site right now:** `https://prasadsawant21-lab.github.io/goat/`  
**Target stack:** Next.js 14 · Supabase · Vercel · Claude Sonnet 4.6 · API-Football  
**Owner:** Prasad — Head of Growth at PIXL Global, Dubai. 12 years GCC performance marketing.

---

## What Already Exists (Do Not Rebuild)

The current `index.html` (4,970 lines) already has:

- ✅ GOAT comparison table — 12 players, 28 metrics, 8 filter datasets
- ✅ Animated GOAT score podium with weighted composite scoring
- ✅ Radar chart (Chart.js) — 6 dimensions
- ✅ 5×6 grid pitch heatmaps — all 12 players
- ✅ FIFA World Cup 2026 live score arena with 5 matches
- ✅ AI commentator — Claude Sonnet 4.6 with match context in system prompt
- ✅ Post-match analysis — timeline, stat bars, POTM card, accuracy ring
- ✅ SWOT analysis per match (both teams)
- ✅ Goal predictions with confidence %
- ✅ Pre-match / half-time / post-match tabs
- ✅ Tactical pitch SVG with player dots and team toggle
- ✅ xG race bars
- ✅ Full mobile responsive (768px / 480px / 360px breakpoints)
- ✅ WC26 group standings (all 12 groups A–L)
- ✅ Score ticker strip
- ✅ League tabs: WC26 · EPL · La Liga · MLS · Champions League

**Design system:** Dark green pitch theme (`#060f06` bg, `#FFD700` gold accent, `Bebas Neue` display font, `Inter` body, `JetBrains Mono` data).

---

## Project Structure to Create

```
goat-platform/
├── CLAUDE.md                          ← this file
├── .env.local                         ← never commit
├── .env.example                       ← commit this
├── next.config.js
├── tailwind.config.js
├── package.json
│
├── app/                               ← Next.js App Router
│   ├── layout.tsx                     ← root layout, fonts, metadata
│   ├── page.tsx                       ← home page (migrated from index.html)
│   ├── globals.css                    ← design tokens + base styles
│   │
│   ├── api/                           ← server-side API routes
│   │   ├── scores/route.ts            ← GET /api/scores?league=wc26
│   │   ├── player/[id]/route.ts       ← GET /api/player/:id
│   │   ├── commentary/route.ts        ← POST /api/commentary (Claude)
│   │   ├── prediction/route.ts        ← POST /api/prediction
│   │   └── webhook/sports/route.ts    ← POST from n8n cron
│   │
│   ├── match/[id]/page.tsx            ← deep dive match page
│   ├── player/[id]/page.tsx           ← player profile page
│   ├── goat/page.tsx                  ← GOAT comparison page
│   └── fantasy/page.tsx              ← fantasy league (Phase 2)
│
├── components/
│   ├── ui/                            ← base components
│   │   ├── Card.tsx
│   │   ├── Badge.tsx
│   │   ├── Tabs.tsx
│   │   └── Spinner.tsx
│   │
│   ├── match/
│   │   ├── ScoreTicker.tsx            ← live score strip at top
│   │   ├── MatchCard.tsx              ← match card in grid
│   │   ├── LiveArena.tsx              ← full match deep-dive panel
│   │   ├── LiveFeed.tsx               ← minute-by-minute events
│   │   ├── TacticalPitch.tsx          ← SVG pitch with heatmap + players
│   │   ├── StatBars.tsx               ← dual-direction stat bars
│   │   ├── XGRace.tsx                 ← expected goals visual
│   │   ├── PostMatch.tsx              ← scoreboard + timeline + POTM
│   │   └── SwotPanel.tsx              ← SWOT analysis cards
│   │
│   ├── goat/
│   │   ├── PodiumGrid.tsx             ← GOAT score ranking
│   │   ├── RadarChart.tsx             ← Chart.js radar
│   │   ├── ComparisonTable.tsx        ← full 28-metric table
│   │   ├── HeatmapCard.tsx            ← individual player heatmap
│   │   └── WeightsPanel.tsx           ← methodology breakdown
│   │
│   └── ai/
│       ├── CommentatorChat.tsx        ← AI chat interface
│       ├── QuickButtons.tsx           ← pre-set question chips
│       └── ThinkingIndicator.tsx      ← animated dots
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts                  ← browser supabase client
│   │   ├── server.ts                  ← server supabase client
│   │   └── types.ts                   ← generated DB types
│   │
│   ├── api/
│   │   ├── sports.ts                  ← API-Football wrapper
│   │   └── claude.ts                  ← Anthropic SDK wrapper
│   │
│   ├── data/
│   │   ├── players.ts                 ← GOAT player stats (migrated from HTML)
│   │   ├── matches.ts                 ← match SWOT + analysis data
│   │   └── weights.ts                 ← GOAT score weight config
│   │
│   └── utils/
│       ├── goat-score.ts              ← weighted score calculator
│       ├── heatmap.ts                 ← heat colour interpolation
│       └── format.ts                  ← number / date formatters
│
├── hooks/
│   ├── useScores.ts                   ← SWR hook for live scores
│   ├── useMatch.ts                    ← match detail + events
│   ├── usePlayer.ts                   ← player stats
│   └── useChat.ts                     ← AI commentary state
│
├── store/
│   └── index.ts                       ← Zustand global store
│
├── types/
│   └── index.ts                       ← shared TypeScript types
│
└── supabase/
    ├── migrations/
    │   └── 001_initial.sql            ← full schema (see below)
    └── seed.sql                       ← seed player + match data
```

---

## Environment Variables

```bash
# .env.local — never commit this file

# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGc...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGc...          # server only

# Anthropic (NEVER expose to browser)
CLAUDE_API_KEY=sk-ant-api03-...

# Sports data
SPORTS_API_KEY=your_api_football_key           # api-football.com
SPORTS_API_HOST=v3.football.api-sports.io

# App
NEXT_PUBLIC_APP_URL=https://yourdomain.com
NEXT_PUBLIC_SITE_NAME=GoatScore
CRON_SECRET=random_string_for_n8n_webhook      # validate n8n calls
```

```bash
# .env.example — commit this
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
CLAUDE_API_KEY=
SPORTS_API_KEY=
SPORTS_API_HOST=v3.football.api-sports.io
NEXT_PUBLIC_APP_URL=
NEXT_PUBLIC_SITE_NAME=GoatScore
CRON_SECRET=
```

---

## Database Schema

Run this in Supabase SQL editor (`supabase/migrations/001_initial.sql`):

```sql
-- Enable extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS vector;

-- ─── MATCHES ───────────────────────────────────────
CREATE TABLE matches (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  external_id     TEXT UNIQUE,                    -- from sports API
  home_team       TEXT NOT NULL,
  away_team       TEXT NOT NULL,
  home_abbr       TEXT NOT NULL,
  away_abbr       TEXT NOT NULL,
  home_score      INT DEFAULT 0,
  away_score      INT DEFAULT 0,
  status          TEXT DEFAULT 'scheduled',        -- scheduled | live | final
  minute          INT,
  kick_off        TIMESTAMPTZ,
  league          TEXT,
  season          TEXT,
  group_name      TEXT,                            -- "Group A", "Group J" etc
  home_prob       FLOAT,
  draw_prob       FLOAT,
  away_prob       FLOAT,
  home_xg         FLOAT DEFAULT 0,
  away_xg         FLOAT DEFAULT 0,
  home_possession INT DEFAULT 50,
  away_possession INT DEFAULT 50,
  stats_json      JSONB DEFAULT '{}',              -- shots, corners, fouls etc
  events_json     JSONB DEFAULT '[]',              -- timeline events array
  lineups_json    JSONB DEFAULT '{}',              -- home/away player arrays
  swot_json       JSONB DEFAULT '{}',              -- pre-built SWOT analysis
  prediction_json JSONB DEFAULT '{}',              -- AI prediction data
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_matches_status ON matches(status);
CREATE INDEX idx_matches_kick_off ON matches(kick_off DESC);
CREATE INDEX idx_matches_league ON matches(league);

-- ─── PLAYERS ───────────────────────────────────────
CREATE TABLE players (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug            TEXT UNIQUE NOT NULL,            -- 'messi', 'ronaldo'
  name            TEXT NOT NULL,
  nation          TEXT,
  club            TEXT,
  color           TEXT,                            -- hex for UI
  abbr            TEXT,                            -- 2-char initials
  goat_score      FLOAT,
  stats_json      JSONB DEFAULT '{}',              -- all 28 metrics per dataset
  heatmap_json    JSONB DEFAULT '{}',              -- 5x6 grid per player
  bio_embedding   vector(1536),                    -- for RAG similarity search
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_players_slug ON players(slug);

-- ─── PROFILES ──────────────────────────────────────
CREATE TABLE profiles (
  id                  UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  username            TEXT UNIQUE,
  display_name        TEXT,
  avatar_url          TEXT,
  favourite_teams     TEXT[] DEFAULT '{}',
  favourite_players   UUID[] DEFAULT '{}',
  pro_subscriber      BOOLEAN DEFAULT FALSE,
  subscription_end    TIMESTAMPTZ,
  predictions_count   INT DEFAULT 0,
  correct_predictions INT DEFAULT 0,
  language            TEXT DEFAULT 'en',           -- 'en' | 'ar'
  created_at          TIMESTAMPTZ DEFAULT NOW(),
  updated_at          TIMESTAMPTZ DEFAULT NOW()
);

-- ─── PREDICTIONS ───────────────────────────────────
CREATE TABLE predictions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES profiles(id) ON DELETE CASCADE,
  match_id        UUID REFERENCES matches(id) ON DELETE CASCADE,
  predicted_home  INT NOT NULL,
  predicted_away  INT NOT NULL,
  points_scored   INT DEFAULT 0,
  is_resolved     BOOLEAN DEFAULT FALSE,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, match_id)
);

CREATE INDEX idx_predictions_user ON predictions(user_id);
CREATE INDEX idx_predictions_match ON predictions(match_id);

-- ─── AI COMMENTARY CACHE ───────────────────────────
CREATE TABLE commentary_cache (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  match_id        UUID REFERENCES matches(id) ON DELETE CASCADE,
  question        TEXT NOT NULL,
  answer          TEXT NOT NULL,
  tokens_used     INT,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_commentary_match ON commentary_cache(match_id);

-- ─── ROW LEVEL SECURITY ────────────────────────────
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE predictions ENABLE ROW LEVEL SECURITY;
ALTER TABLE commentary_cache ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public matches" ON matches FOR SELECT USING (true);
CREATE POLICY "Public players" ON players FOR SELECT USING (true);
CREATE POLICY "Own profile" ON profiles FOR ALL USING (auth.uid() = id);
CREATE POLICY "Own predictions" ON predictions FOR ALL USING (auth.uid() = user_id);
CREATE POLICY "Public commentary" ON commentary_cache FOR SELECT USING (true);

-- ─── UPDATED_AT TRIGGER ────────────────────────────
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN NEW.updated_at = NOW(); RETURN NEW; END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER matches_updated_at BEFORE UPDATE ON matches
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER players_updated_at BEFORE UPDATE ON players
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER profiles_updated_at BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

---

## API Routes — Exact Contracts

### GET `/api/scores`

```typescript
// Query params: ?league=wc26&status=live
// Returns: Match[] from Supabase, falls back to sports API if stale

// Response shape
{
  league: string
  matches: {
    id: string
    home: string
    away: string
    homeAbbr: string
    awayAbbr: string
    homeScore: number
    awayScore: number
    status: 'scheduled' | 'live' | 'final'
    minute: number | null
    kickOff: string
    prob: { home: number; draw: number; away: number }
  }[]
  updatedAt: string
}

// Cache strategy: Redis TTL 60s for live, 300s for scheduled/final
```

### POST `/api/commentary`

```typescript
// Body
{
  matchId: string
  messages: { role: 'user' | 'assistant'; content: string }[]
  matchContext: string   // pre-built context string for this match
}

// Response
{
  reply: string
  tokensUsed: number
}

// Implementation — CRITICAL security rules:
// 1. NEVER expose CLAUDE_API_KEY to browser
// 2. Rate limit: 20 requests per user per hour (use Upstash Redis)
// 3. Max context: 4000 tokens (trim older messages if needed)
// 4. Cache identical questions for same match (commentary_cache table)
```

### POST `/api/webhook/sports`

```typescript
// Called by n8n every 5 minutes
// Validates: Authorization: Bearer ${process.env.CRON_SECRET}

// Body: raw API-Football response
// Action: upsert to matches table

// n8n workflow config:
// Trigger: Cron (every 5 min)
// Node 1: HTTP Request → GET https://v3.football.api-sports.io/fixtures?live=all
// Node 2: HTTP Request → POST https://yourdomain.com/api/webhook/sports
//           Headers: Authorization: Bearer {{$env.CRON_SECRET}}
//           Body: {{$json}}
```

---

## Key Components — Implementation Notes

### `TacticalPitch.tsx`

This is the most complex component. Migrated from the HTML version.

```typescript
// Props
interface TacticalPitchProps {
  homeHeat: number[][]    // 5 cols × 6 rows, values 0–1
  awayHeat: number[][]
  homePlayers: Player[]
  awayPlayers: Player[]
  homeColor: string       // hex
  awayColor: string       // hex
  view: 'home' | 'away' | 'both'
  onViewChange: (v: 'home' | 'away' | 'both') => void
}

// The pitch is SVG, viewBox="0 0 280 420"
// Attacking end at BOTTOM of portrait pitch
// 5 columns (left→right), 6 rows (own half top → attacking box bottom)
// Heat colour: dark green (#081908) → yellow-green → bright yellow (#FFEB14)
// Player positions mapped by role: GK, CB, LB, RB, CM, DM, CAM, LW, RW, CF, ST
// Player dots: 12px radius circles with 2-char initials
```

### `CommentatorChat.tsx`

```typescript
// State
const [messages, setMessages] = useState<ChatMessage[]>([])
const [isLoading, setIsLoading] = useState(false)

// IMPORTANT: Call /api/commentary (server route), not Anthropic directly
// The browser must NEVER see the Claude API key

// Opening message: generate from match state (status, score, context)
// Quick questions: pre-defined per match (5 chips below input)
// Conversation history: kept in component state, sent with each request
// Rate limiting: show "slow down" message if 429 received from API

// Thinking indicator: 3 animated dots, 400ms stagger
// Message animation: slide in from bottom, 200ms ease
```

### `useScores.ts`

```typescript
import useSWR from 'swr'

export function useScores(league: string) {
  const { data, error, isLoading } = useSWR(
    `/api/scores?league=${league}`,
    fetcher,
    {
      refreshInterval: 60000,   // refresh every 60s
      revalidateOnFocus: true,
      revalidateOnReconnect: true,
    }
  )
  return { scores: data, error, isLoading }
}
// Live matches should refresh every 30s (pass refreshInterval as prop)
// Scheduled/final: every 5 minutes is fine
```

---

## Design System — Exact Values

These must be preserved exactly from the original HTML:

```css
/* globals.css */
:root {
  --bg:          #060f06;
  --bg-card:     rgba(255, 255, 255, 0.04);
  --border:      rgba(255, 255, 255, 0.08);
  --gold:        #FFD700;
  --gold-dim:    #b8960a;
  --silver:      #C0C0C0;
  --bronze:      #CD7F32;
  --white:       #f4f4f4;
  --muted:       #9aa3a8;
  --radius:      12px;

  /* Player colours */
  --messi:       #5b9bd5;
  --ronaldo:     #e05252;
  --neymar:      #4caf6e;
  --mbappe:      #26c6da;
  --lewandowski: #ff9800;
  --suarez:      #9c27b0;
  --benzema:     #f4c542;
  --salah:       #e8403a;
  --vinicius:    #ffd700;
  --haaland:     #6bc5f8;
  --dybala:      #c77dff;
  --modric:      #80ffdb;
}
```

```typescript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        bg: '#060f06',
        gold: '#FFD700',
        'gold-dim': '#b8960a',
        muted: '#9aa3a8',
        card: 'rgba(255,255,255,0.04)',
      },
      fontFamily: {
        display: ['Bebas Neue', 'sans-serif'],
        body:    ['Inter', 'sans-serif'],
        mono:    ['JetBrains Mono', 'monospace'],
      },
    },
  },
}
```

---

## GOAT Score Algorithm

This must be preserved exactly — it powers the entire comparison engine.

```typescript
// lib/utils/goat-score.ts

export const WEIGHTS = {
  Goalscoring: {
    pct: 28,
    items: {
      'Career goals total':  { key: 'goals_total',    w: 10 },
      'Goals per 90 min':    { key: 'goals_per90',    w: 8  },
      'Hat-tricks':          { key: 'hattricks',      w: 5  },
      'Free-kick goals':     { key: 'free_kick_goals', w: 5 },
    }
  },
  Creation: {
    pct: 16,
    items: {
      'Total assists':       { key: 'assists_total',      w: 10 },
      'Goal contributions':  { key: 'goal_contributions', w: 6  },
    }
  },
  Trophies: {
    pct: 28,
    items: {
      'Champions League':    { key: 'cl_titles',          w: 8 },
      'League titles':       { key: 'league_titles',      w: 6 },
      'World Cup':           { key: 'world_cup',          w: 8 },
      'Copa/Euro':           { key: 'copa_america_euro',  w: 4 },
      'Total trophies':      { key: 'total_trophies',     w: 2 },
    }
  },
  Awards: {
    pct: 16,
    items: {
      "Ballon d'Or":         { key: 'ballon_dor',              w: 8 },
      'Golden Boot':         { key: 'golden_boot_league',      w: 3 },
      'FIFA Best':           { key: 'best_fifa',               w: 3 },
      'UCL Top Scorer':      { key: 'champions_lg_topscorer',  w: 2 },
    }
  },
  BigGames: {
    pct: 12,
    items: {
      'KO Stage Goals':      { key: 'big_game_goals', w: 8 },
      'MVPs won':            { key: 'mvp_awards',     w: 4 },
    }
  },
}

export function computeGoatScore(
  playerIndex: number,
  dataset: Record<string, number[]>
): number {
  let total = 0
  let wSum = 0
  for (const cat of Object.values(WEIGHTS)) {
    for (const item of Object.values(cat.items)) {
      const vals = dataset[item.key]
      if (!vals) continue
      const max = Math.max(...vals)
      const norm = max > 0 ? (vals[playerIndex] / max) * 100 : 0
      total += norm * item.w
      wSum += item.w
    }
  }
  return wSum > 0 ? Math.round((total / wSum) * 10) / 10 : 0
}
```

---

## Player Data — All 12 Players

```typescript
// lib/data/players.ts
// Migrate exactly from HTML — all stats arrays indexed [0..11]:
// [messi, ronaldo, neymar, mbappe, lewandowski, suarez,
//  benzema, salah, vinicius, haaland, dybala, modric]

export const PLAYERS = [
  { id: 'messi',       name: 'Messi',       nation: 'Argentina', color: '#5b9bd5', abbr: 'LM'  },
  { id: 'ronaldo',     name: 'Ronaldo',      nation: 'Portugal',  color: '#e05252', abbr: 'CR'  },
  { id: 'neymar',      name: 'Neymar',       nation: 'Brazil',    color: '#4caf6e', abbr: 'NJ'  },
  { id: 'mbappe',      name: 'Mbappé',       nation: 'France',    color: '#26c6da', abbr: 'KM'  },
  { id: 'lewandowski', name: 'Lewandowski',  nation: 'Poland',    color: '#ff9800', abbr: 'RL'  },
  { id: 'suarez',      name: 'Suárez',       nation: 'Uruguay',   color: '#9c27b0', abbr: 'LS'  },
  { id: 'benzema',     name: 'Benzema',      nation: 'France',    color: '#f4c542', abbr: 'KB'  },
  { id: 'salah',       name: 'Salah',        nation: 'Egypt',     color: '#e8403a', abbr: 'MS'  },
  { id: 'vinicius',    name: 'Vinicius Jr',  nation: 'Brazil',    color: '#ffd700', abbr: 'VJ'  },
  { id: 'haaland',     name: 'Haaland',      nation: 'Norway',    color: '#6bc5f8', abbr: 'EH'  },
  { id: 'dybala',      name: 'Dybala',       nation: 'Argentina', color: '#c77dff', abbr: 'PD'  },
  { id: 'modric',      name: 'Modrić',       nation: 'Croatia',   color: '#80ffdb', abbr: 'LM2' },
]
```

---

## Claude AI Commentary — System Prompt Template

```typescript
// lib/api/claude.ts

export function buildSystemPrompt(matchContext: string): string {
  return `You are an expert football AI commentator and analyst with deep tactical knowledge.
You are analysing a specific live match and providing commentary, tactical breakdowns, and insights.

CURRENT MATCH CONTEXT:
${matchContext}

YOUR PERSONALITY:
- Passionate, knowledgeable, conversational
- Direct and opinionated — backed by data, not wishy-washy
- Use football terminology naturally (xG, pressing trap, half-spaces, PPDA)
- Maximum 3 sentences per response unless explaining tactics in detail
- Occasional emojis are fine — don't overdo it
- Never say "I don't have real-time data" — use the context provided above
- If asked about something not in context, make an educated analysis based on what you know
- For Arabic questions: respond in Arabic with the same analytical depth

RESPONSE RULES:
- Short questions → short punchy answers (1-2 sentences)
- Tactical questions → up to 5 sentences with specific details
- "Who is the GOAT?" questions → reference the weighted GOAT score model
- Always name specific players, never say "a player"
- If predicting → give a specific scoreline with reasoning`
}

export async function getCommentary(
  messages: ChatMessage[],
  matchContext: string
): Promise<string> {
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.CLAUDE_API_KEY!,
      'anthropic-version': '2023-06-01',
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-6',
      max_tokens: 1000,
      system: buildSystemPrompt(matchContext),
      messages,
    }),
  })

  if (!response.ok) {
    throw new Error(`Claude API error: ${response.status}`)
  }

  const data = await response.json()
  return data.content[0].text
}
```

---

## n8n Workflow — Live Score Polling

Set this up in your existing n8n instance at PIXL Global:

```
Workflow: "Football Score Poller"

Node 1: Schedule Trigger
  → Every 5 minutes (during match hours: 16:00–24:00 GST)
  → Every 30 minutes (during quiet hours)

Node 2: HTTP Request
  → Method: GET
  → URL: https://v3.football.api-sports.io/fixtures?live=all
  → Headers:
      x-rapidapi-host: v3.football.api-sports.io
      x-rapidapi-key: {{ $env.SPORTS_API_KEY }}

Node 3: HTTP Request
  → Method: POST
  → URL: {{ $env.PLATFORM_URL }}/api/webhook/sports
  → Headers:
      Authorization: Bearer {{ $env.CRON_SECRET }}
      Content-Type: application/json
  → Body: {{ JSON.stringify($json) }}

Node 4: IF (on error)
  → Send Slack alert to #platform-alerts
  → Log to Supabase errors table
```

---

## Deployment — Step by Step

```bash
# 1. Install dependencies
npm install

# 2. Set up Supabase
#    → supabase.com → New project → "goat-platform"
#    → SQL Editor → run supabase/migrations/001_initial.sql
#    → Run supabase/seed.sql to populate players

# 3. Configure environment
cp .env.example .env.local
# Fill in all values

# 4. Run locally
npm run dev
# → http://localhost:3000

# 5. Deploy to Vercel
npx vercel
# Follow prompts → connect GitHub repo → add env vars in Vercel dashboard

# 6. Set up custom domain
#    → Vercel dashboard → Domains → add yourdomain.com
#    → Cloudflare → DNS → CNAME → cname.vercel-dns.com

# 7. Configure n8n
#    → Add PLATFORM_URL env var → your Vercel URL
#    → Activate workflow → verify first webhook hit in Supabase
```

---

## What to Build First (Priority Order)

```
WEEK 1 — Backend foundation
  [ ] Scaffold Next.js project with TypeScript + Tailwind
  [ ] Set up Supabase project + run migration SQL
  [ ] Create /api/scores route with API-Football integration
  [ ] Create /api/commentary route (move Claude key server-side)
  [ ] Set up n8n webhook receiver (/api/webhook/sports)
  [ ] Deploy to Vercel

WEEK 2 — Migrate existing features
  [ ] Extract player data from index.html → lib/data/players.ts
  [ ] Build CommentatorChat component (calls /api/commentary)
  [ ] Build TacticalPitch component (SVG heatmap + player dots)
  [ ] Build MatchCard + LiveArena components
  [ ] Build GOAT comparison table + radar chart
  [ ] Connect everything to Supabase via useScores / useMatch hooks

WEEK 3 — Auth + user features
  [ ] Add Supabase Auth (Google OAuth)
  [ ] Build profile page + favourite teams
  [ ] Build prediction submit form
  [ ] Add RLS policies + test security

WEEK 4 — Polish + launch
  [ ] Arabic language toggle (RTL layout + Claude Arabic prompts)
  [ ] SEO metadata + sitemap
  [ ] Error boundaries + loading states
  [ ] Performance audit (Lighthouse target: >90 mobile)
  [ ] Soft launch to friends via updated GitHub Pages URL
```

---

## Do Nots — Hard Rules

```
❌ NEVER put CLAUDE_API_KEY in any client-side code
❌ NEVER put SUPABASE_SERVICE_ROLE_KEY in any client-side code
❌ NEVER commit .env.local
❌ DO NOT rebuild the GOAT score algorithm — it's calibrated, preserve exactly
❌ DO NOT change the colour scheme — the dark green + gold is the brand
❌ DO NOT use serif fonts — Bebas Neue + Inter + JetBrains Mono only
❌ DO NOT break mobile responsiveness — test at 360px, 480px, 768px
❌ DO NOT make API calls to sports APIs from the browser — server routes only
❌ DO NOT call Claude directly from browser components — always use /api/commentary
```

---

## Do Dos — Must Follow

```
✅ Use TypeScript throughout — strict mode
✅ Use SWR for all data fetching with appropriate refresh intervals
✅ Cache API-Football responses in Upstash Redis (60s for live, 300s for rest)
✅ Handle loading + error states in every component
✅ Preserve the exact GOAT score weighting from lib/utils/goat-score.ts
✅ Keep the SVG pitch portrait orientation (attacking end at bottom)
✅ Rate limit the /api/commentary route (20 req/user/hour)
✅ Use Supabase Row Level Security on all user tables
✅ Test Arabic RTL layout when adding text — use dir="rtl" on root for AR
✅ Log errors to Sentry (add as first priority after core features work)
```

---

## Reference Files

| File | Purpose |
|------|---------|
| `index.html` | Original single-file app — source of truth for all features |
| `supabase/migrations/001_initial.sql` | Full DB schema |
| `supabase/seed.sql` | Player + match seed data |
| `.env.example` | Required environment variables |
| `lib/data/players.ts` | All 12 player stats (migrated from HTML) |
| `lib/utils/goat-score.ts` | GOAT score algorithm — do not modify |

---

## Questions? Context for Claude Code

- **Owner:** Prasad Sawant, Dubai — `prasadsawant21-lab` on GitHub
- **Current repo:** `https://github.com/prasadsawant21-lab/goat`
- **Live site:** `https://prasadsawant21-lab.github.io/goat/`
- **Stack preference:** TypeScript, Next.js App Router, Tailwind, Supabase
- **AI model:** Claude Sonnet 4.6 (not Opus — cost efficiency matters)
- **Primary market:** MENA football fans (English first, Arabic second)
- **Monetisation:** Freemium — free tier + $4.99/mo Pro

*If something is ambiguous, default to: simplest implementation that works, TypeScript strict, mobile-first, dark theme.*
