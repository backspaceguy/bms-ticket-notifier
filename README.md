# BMS Ticket Notifier

Automatically monitors [BookMyShow](https://in.bookmyshow.com) for ticket availability and sends you an email alert when something changes.

Runs every n minutes via GitHub Actions. (You can set it via cron jobs, tho GH Actions doesn't kick in accurately)

## How It Works

1. Fetches showtimes from the BookMyShow API for one or more movie URLs
2. Compares results with the previous check (stored per-movie in `bms_state.json`)
3. Sends an HTML email via [Resend](https://resend.com) for each movie with changes (new shows, dates opening, availability updates)

## Setup

### 1. Fork this repository

### 2. Set GitHub Secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|--------|-------------|
| `RESEND_API_KEY` | API key from [resend.com](https://resend.com) |
| `RESEND_FROM_EMAIL` | Email address to send notifications (use your own domain email or anything@resend.dev) |
| `RESEND_TO_EMAIL` | Email address to receive notifications |

### 3. Set GitHub Variables

Go to **Settings → Secrets and variables → Actions → Variables** and add:

| Variable | Description | Example |
|----------|-------------|---------|
| `BMS_URL` | BookMyShow ticket page URL(s). List multiple movies one per line (or `;`-separated) to watch several at once. | `https://in.bookmyshow.com/movies/chennai/.../ET00123456` |
| `BMS_DATES` | Dates to monitor (YYYYMMDD, comma-separated). Leave empty to auto-detect from URL. | `20260318,20260319` |
| `BMS_THEATRE` | Filter by theatre name (substring match, comma-separated) | `PVR,Sathyam` |
| `BMS_TIME` | Filter by time period (comma-separated) | `evening,night` |

**Watching multiple movies:** put one URL per line in `BMS_URL`. `BMS_DATES` / `BMS_THEATRE` / `BMS_TIME` each apply to *every* movie if given a single value (as above), or can be set per-movie by giving one value per line, in the same order as `BMS_URL`.

```
BMS_URL:
https://in.bookmyshow.com/movies/chennai/movie-one/ET00111111
https://in.bookmyshow.com/movies/chennai/movie-two/ET00222222

BMS_THEATRE: PVR   # applies to both movies above
```

**Time periods:** `morning` (6–12), `afternoon` (12–16), `evening` (16–19), `night` (19–24)

**Premium formats (IMAX, 4DX, etc.) are not a theatre filter.** BookMyShow gives each format its own ticket page and event code — `BMS_THEATRE=IMAX` will *not* find IMAX shows, since "IMAX" doesn't appear in the venue name. Instead, open the movie on BookMyShow, pick the IMAX (or 4DX/Screen X/...) format from the language/format selector, and use *that* page's URL/event code in `BMS_URL`. You'll recognize it by a URL slug ending in e.g. `-imax-2d`.

### 4. Trigger the workflow

Go to **Actions → BMS Ticket Checker** and click **Run workflow**, or wait for it to run automatically every 30 minutes.

## Local Usage

Requires Python 3.14+ and [uv](https://docs.astral.sh/uv/).

```bash
uv sync --frozen

export BMS_URL="https://in.bookmyshow.com/movies/chennai/.../ET00123456"
export BMS_DATES="20260318,20260319"
export BMS_THEATRE="PVR"
export BMS_TIME="evening,night"
export RESEND_API_KEY="re_..."
export RESEND_TO_EMAIL="you@example.com"
export RESEND_TO_EMAIL="python@resend.dev"

uv run main.py
```

## Notifications

You'll receive an email when:
- A new showtime is added
- A date opens for booking
- Seat availability changes (e.g. sold out → available)

Emails show a summary of what changed and the current status of all monitored shows, grouped by theatre.
