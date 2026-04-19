# Enhancement: Live Artist Earnings Estimator

## What was built

A section at the bottom of the app that let users search for any artist by
name on Spotify and see an estimated monthly earnings range, derived from
Spotify's popularity score (0–100).

### How it worked
1. User enters Spotify API Client ID + Secret (stored in localStorage)
2. App authenticates via Client Credentials flow (`POST /api/token`)
3. Searches for artist via `GET /v1/search?type=artist`
4. Fetches full artist object via `GET /v1/artists/{id}` for popularity + followers
5. Maps popularity score → estimated monthly stream range (non-linear bands)
6. Displays: artist photo, follower count, score bar, earnings range, "enough
   to afford X × [selected item] per month"

### Popularity → streams mapping
Non-linear banding, each 10-point range roughly doubles the prior:

| Score | Est. monthly streams |
|-------|----------------------|
| 90–100 | 200M – 800M |
| 80–89  | 60M – 200M  |
| 70–79  | 15M – 60M   |
| 60–69  | 4M – 15M    |
| 50–59  | 1M – 4M     |
| 40–49  | 250K – 1M   |
| 30–39  | 60K – 250K  |
| 20–29  | 12K – 60K   |
| 10–19  | 2K – 12K    |
| 0–9    | 100 – 2K    |

Earnings = stream_estimate × selected platform rate (e.g. $0.004 for Spotify).

## Why it was removed

Spotify's February 2026 API changes stripped `popularity` and `followers`
fields from all artist endpoints (`/v1/artists/{id}` and `/v1/search`).
The `top-tracks` endpoint also returns 403 in Development Mode.

Reference: https://developer.spotify.com/blog/2026-02-06-update-on-developer-access-and-platform-security

Additional constraints introduced:
- Spotify Premium required to create a developer app
- One Client ID per developer, max 5 authorized users
- Restricted endpoint access in Development Mode

## What's needed to revive it

- A data source that exposes artist popularity or stream counts. Options:
  - Wait for Spotify to restore fields (unlikely short-term)
  - Chartmetric or Soundcharts API (paid, ~$100–500/mo) — most accurate
  - Last.fm API (free) — scrobble counts only, platform-agnostic, skews indie
  - YouTube Data API (free) — view counts on music videos only
- If using a paid source, a lightweight backend proxy to hold credentials
  rather than exposing them client-side

## GitHub issue
https://github.com/mchegge/streaming-reality-check/issues/1
