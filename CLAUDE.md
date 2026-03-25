# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SUMA+ Status Dashboard — a single-page, static HTML status dashboard that monitors the health of external SaaS services used by the SUMA+ team. Everything (HTML, CSS, JS) lives in one file: `index.html`. No build tools, no dependencies, no server required.

## How to Run

Open `index.html` directly in a browser, or serve it with any static file server:
```
python3 -m http.server 8080
```

## Architecture

### Data Flow
1. On load (and every 5 minutes), `refreshData()` fetches status from each service's API
2. Requests go direct first; on failure, fall through two CORS proxies (`corsproxy.io`, `allorigins.win`)
3. Each service type has its own parser: `parseStatuspageData` (Atlassian Statuspage format), `parseSlackStatus`, `parseGoogleStatus`
4. Parsed state is stored in `serviceStates` (keyed by service id) and rendered into three UI sections

### Monitored Services (SERVICES array)
Mixpanel, Slack, Atlassian (Jira/Confluence), Claude (Anthropic), Google Apps, HubSpot. Each entry defines `type` which determines the parser used: `statuspage`, `slack`, or `google`.

### UI Sections
- **Header**: donut chart showing operational ratio, last-update timestamp, 5-minute countdown ring
- **Overview cards**: one per service with 30-day uptime bar (simulated history), status badge
- **Component matrix**: per-service breakdown of sub-components and their status
- **Incidents panel**: aggregated recent incidents from all services, sorted by recency

### CORS Proxy Fallback Chain
`fetchWithProxy(url)` tries a direct fetch first, then falls through `CORS_PROXIES` array in order. If all fail, returns `null` and the service renders with fallback "operational" state (`isFallback: true`). Timeout per attempt: 8 seconds.

### Adding a New Service
1. Add entry to `SERVICES` array with `id`, `name`, `sub`, `url`, `apiUrl`, `type`, `color`, `letter`
2. If the API format isn't `statuspage`, `slack`, or `google`, create a new `parse*` function
3. If the service needs component fallbacks, add to `FALLBACKS` object
4. The UI auto-renders — no template changes needed

### Key Globals
- `serviceStates` — object mapping service id to parsed status data
- `uptimeHistory` — object mapping service id to 30-element array of day statuses (`ok`/`warn`/`err`/`none`)
- `REFRESH_INTERVAL` — 300 seconds (5 min) auto-refresh cycle

## Design System
- **Brand colors**: defined as CSS custom properties (e.g., `--azul-suma: #0087CD`, `--azul-oscuro: #1C4070`)
- **Fonts**: Poppins (headings/UI) + Roboto (body), loaded from Google Fonts
- **Language**: all UI text is in Spanish (es-CL locale)
- **Responsive**: CSS grid breakpoints at 1100px, 860px, 520px
