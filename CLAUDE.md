# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# "Yelling at the Cloud" — Implementation Spec

## Project Overview

A static HTML/JS website that checks the live status of major cloud/SaaS providers and displays the results using Abe Simpson illustrations.

- **If any tracked services are down** → Abe shakes his fist at the cloud (angry pose), with affected service icons displayed inside the cloud
- **If all services are operational** → Abe sleeps peacefully (sleeping pose)
- Deployable to GitHub Pages with zero build step, zero backend

---

## Technical Stack

- **Pure static site**: HTML, inline CSS, inline JS
- **External SVG images**: `yelling.svg` and `asleep.svg` for Abe poses
- **Inline SVG icons**: Black service logos embedded in JavaScript
- **GitHub Pages compatible**: all fetches are client-side
- **Auto-refresh**: poll all status APIs every 60 seconds
- **Streaming updates**: UI updates as each service responds (no waiting for all)

---

## File Structure

```
index.html    ← HTML, CSS, JS all inline
yelling.svg   ← Abe yelling at cloud (angry pose)
asleep.svg    ← Abe sleeping (peaceful pose)
CLAUDE.md     ← This file
```

---

## Monitored Services

| Service     | Type       | API URL | Notes |
|-------------|------------|---------|-------|
| Anthropic   | statuspage | `https://status.anthropic.com/api/v2/summary.json` | Requires CORS proxy |
| AWS         | rss        | `https://status.aws.amazon.com/rss/all.rss` | Via CORS proxy, filters unresolved issues |
| Cloudflare  | statuspage | `https://www.cloudflarestatus.com/api/v2/summary.json` | Direct fetch |
| GitHub      | statuspage | `https://www.githubstatus.com/api/v2/summary.json` | Direct fetch |
| OpenAI      | statuspage | `https://status.openai.com/api/v2/summary.json` | Direct fetch |

### Statuspage API Pattern
```js
const data = await res.json();
const issues = data.components.filter(c => c.status !== 'operational');
const hasIssues = issues.length > 0 || (data.incidents?.length > 0);
```

### AWS RSS Pattern
- Fetched via `allorigins.win` CORS proxy
- May return base64-encoded content (handled automatically)
- Filters by title patterns: "Service disruption", "Service impact"
- Excludes resolved items

---

## Service Registry

```js
const SERVICES = [
  {
    id: 'github',
    name: 'GitHub',
    type: 'statuspage',
    url: 'https://www.githubstatus.com/api/v2/summary.json',
    statusPageUrl: 'https://www.githubstatus.com'
  },
  {
    id: 'anthropic',
    name: 'Anthropic',
    type: 'statuspage',
    url: 'https://status.anthropic.com/api/v2/summary.json',
    statusPageUrl: 'https://status.anthropic.com',
    useProxy: true  // Requires CORS proxy
  },
  // ... etc
];
```

---

## UI Layout

```
┌─────────────────────────────────────────┐
│                                         │
│         [cloud with service icons]      │
│              [ABE SVG]                  │
│                                         │
│  ⚠️ AWS · Anthropic · GitHub · OpenAI   │  ← Combined service list
│                                         │
│       Last checked: 14 seconds ago      │
└─────────────────────────────────────────┘
```

### Service List Features
- **Single combined line** showing all monitored services
- **Down services appear first** with ⚠️ emoji and red text
- **All services are clickable links** to their status pages
- **Streaming updates**: services appear as their status is fetched

### Cloud Icons
- Service icons appear inside the cloud when down
- Black SVG icons with drop shadow (no white background)
- Single horizontal row layout
- Icons link to service status pages

---

## Aesthetic Direction

- **Background**: Deep navy (`#1a1a2e`) with subtle radial gradients
- **Font**: `Bangers` for display, `Space Mono` for status text
- **Colors**:
  - Down services: Red (`#f87171`)
  - Operational: Green (`#4ade80`)
  - Unknown: Gray (`#888`)
- **Animations**: Service icons fade in when detected

---

## Console Features

On page load:
```
☁️ Yelling at the Cloud
Vibe coded for fun by Art https://github.com/artkay
```

After each status check:
```
🌩️ Service Status Report
  ✅ GitHub: No issues detected
  ❌ AWS: Service impact: Increased Connectivity Issues
  ✅ OpenAI: No issues detected
  ...
```

---

## Error Handling

- **CORS issues**: Use `allorigins.win` proxy for services that need it
- **Proxy returns base64**: Automatically detected and decoded
- **Fetch failures**: Mark service as "unknown", don't count as down
- **All fetches fail**: Show "Even the status pages are down. Abe is unreachable."
- **Every fetch wrapped in try/catch**: Page never crashes

---

## GitHub Pages Deployment

1. Push all files to repo
2. Enable GitHub Pages from settings → branch `main`, root `/`
3. Done — accessible at `https://<username>.github.io/yellingatthe.cloud/`

No CI, no build step, no `package.json` needed.

---

## Future Ideas (out of scope for v1)

- Let users toggle which services they care about (saved to `localStorage`)
- Add more services (Stripe, PagerDuty, etc.)
- Sound effect: Abe grumbling when a new outage is detected (opt-in)
