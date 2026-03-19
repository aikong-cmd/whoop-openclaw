# WHOOP Health Data Sync 🏋️

[中文文档 / Chinese README](README_CN.md)

Sync WHOOP wearable data to markdown files for AI-powered health insights.

Pure Python, zero dependencies (stdlib + curl only). 10-minute setup, authorize once, runs forever.

## Features

| Category | Data |
|----------|------|
| **Recovery** | Score (🔴🟡🟢), HRV (RMSSD), Resting HR, SpO2, Skin Temp |
| **Sleep** | Performance/Efficiency/Consistency, Stages (Light/Deep/REM/Awake), Respiratory Rate, Sleep Need, Balance |
| **Day Strain** | Strain (0-21), Calories (kJ + kcal), Avg/Max HR |
| **Workouts** | Sport, Duration, Strain, Calories, HR, HR Zones, Distance, Elevation |
| **Weekly** | Averages for recovery/HRV/RHR, sleep, strain, workout count |

## Install

### Option A: OpenClaw (one command)

```bash
clawhub install whoop
```

### Option B: Manual

```bash
git clone https://github.com/aikong-cmd/whoop-openclaw.git
cp -r whoop-openclaw ~/.openclaw/workspace/skills/whoop
```

## Setup

### 1. Create a WHOOP Developer App

1. Go to [developer-dashboard.whoop.com](https://developer-dashboard.whoop.com/)
2. Sign in with your WHOOP account
3. Click **Create Application**
   - **Name**: anything (e.g. "AI Health Sync")
   - **Redirect URI**: `http://localhost:9527/callback`
   - **Scopes**: select all `read:*` scopes + `offline`
4. Note your **Client ID** and **Client Secret**

### 2. Store Credentials

**Environment Variables (simplest):**

```bash
export WHOOP_CLIENT_ID="your-client-id"
export WHOOP_CLIENT_SECRET="your-client-secret"
```

**Or 1Password (for OpenClaw users):**

Create a Login item named `whoop` (username = Client ID, password = Client Secret).

### 3. Authorize (one-time)

#### 🖥️ Local Machine

```bash
python3 scripts/auth.py
```

Opens URL → authorize in browser → callback auto-caught → done ✅

#### 🌐 Remote Server (headless)

```bash
# Get the auth URL
python3 scripts/auth.py --print-url

# Open URL in local browser, authorize
# Browser redirects to localhost:9527 (won't load — that's fine)
# Copy the full URL from address bar, then:

python3 scripts/auth.py --callback-url "http://localhost:9527/callback?code=xxx&state=yyy"
```

**Authorize once, runs forever.** Tokens auto-refresh via the `offline` scope.

## Usage

```bash
python3 scripts/sync.py              # Sync today
python3 scripts/sync.py --days 7     # Last 7 days
python3 scripts/sync.py --weekly     # Weekly summary
python3 scripts/sync.py --date 2026-03-07  # Specific date
```

Output: `~/.openclaw/workspace/health/whoop-YYYY-MM-DD.md`

## Daily Auto-Sync (OpenClaw Cron)

```bash
openclaw cron add \
  --name whoop-daily \
  --schedule "0 10 * * *" \
  --timezone Asia/Shanghai \
  --task "Run: python3 ~/.openclaw/workspace/skills/whoop/scripts/sync.py --days 2. Then read the generated markdown files and send me the latest day's report."
```

## Output Example

```markdown
# WHOOP — 2026-03-09

## Recovery 🟢
- **Recovery Score**: 66%
- **HRV (RMSSD)**: 41.4 ms
- **Resting HR**: 62.0 bpm
- **SpO2**: 96.3%
- **Skin Temp**: 33.7°C

## Sleep
- **Performance**: 🟡 61%
- **Total in Bed**: 5h47m
- **Stages**: Light 2h08m | Deep 1h25m | REM 1h38m | Awake 35m
- **Sleep Need**: 9h41m
- **Balance**: 4h29m deficit

## Day Strain
- **Strain**: 0.1 / 21.0
- **Calories**: 2233 kJ (534 kcal)

## Workouts
- Walking · 16m28s · Strain 4.9 · Avg HR 114 bpm
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `No tokens found` | Run `auth.py` first |
| `Token refresh failed (403)` | Re-run `auth.py` to re-authorize |
| `error code: 1010` | Cloudflare block — script uses curl to bypass |
| `No data for date` | WHOOP sleep finalizes after waking; try later |
| Port 9527 in use | `kill $(lsof -ti:9527)` then retry |

## File Structure

```
whoop/
├── SKILL.md              # AI agent instructions
├── README.md             # English docs (this file)
├── README_CN.md          # Chinese docs
├── scripts/
│   ├── auth.py           # OAuth authorization
│   └── sync.py           # Data sync + weekly reports
└── data/
    ├── tokens.json       # OAuth tokens (auto-refreshed, gitignored)
    └── .gitignore
```

## Requirements

- Python 3.10+
- curl
- Active WHOOP membership

## License

MIT
