# Log Analyzer Pro

Real-time web traffic analysis dashboard with geographic visualization, traffic classification, and threat detection.

![Dashboard](https://img.shields.io/badge/stack-HTML%20%2B%20JS-blue) ![License](https://img.shields.io/badge/license-MIT-green)

## Features

- **Interactive Map** -- Leaflet-based world map with per-session markers color-coded by traffic type
- **Traffic Classification** -- Automatic categorization into legitimate, cloud, hosting, bot, and malicious traffic
- **Scanner Detection** -- Pattern-matching engine flags requests to well-known exploit paths (WordPress probing, `.env` harvesting, path traversal, PHP shells)
- **Notable Organizations** -- Identifies visitors by rDNS and ASN, grouped into cloud providers vs other hosting
- **Country Blocking Overlay** -- Visual overlay on the map for blocked countries with per-country IP and request stats
- **Live Feed** -- Scrollable real-time traffic table with IP reputation links
- **Playback** -- Timeline scrubber with adjustable speed to replay traffic patterns

## Quick Start

```bash
# Clone the repo
git clone https://github.com/your-org/log-analyzer.git
cd log-analyzer

# Serve locally (Python)
cd html
python3 -m http.server 8080

# Open http://localhost:8080
```

The dashboard expects a `data.json` file in the `html/` directory. See [Data Format](#data-format) below.

## Architecture

```
html/
  index.html          # Dashboard shell, modals, Leaflet/Chart.js setup
  app.js              # All application logic (~600 lines)
  styles.css          # Dark theme styles
  countries.geojson   # World borders for blocked country overlay
  data.json           # Session data (gitignored, you provide this)
```

Zero backend dependencies. The dashboard is a static site that reads a single JSON file.

## Data Format

The dashboard consumes a `data.json` file with three top-level keys:

### `sessions` (array)

Each session represents a cluster of requests from one IP:

```json
{
  "origin_ip": "203.0.113.42",
  "edge_ip": "203.0.113.42",
  "geo": {
    "city": "Frankfurt am Main",
    "country": "Germany",
    "country_code": "DE",
    "hostname": "example.host.com",
    "asn": 24940,
    "asn_name": "Hetzner Online GmbH",
    "is_bot": false,
    "is_verified_bot": false,
    "is_cloud": false,
    "is_hosting": true,
    "lat": 50.1109,
    "lon": 8.6821
  },
  "first_seen": 1711800000.0,
  "last_seen": 1711800060.0,
  "intent": "Passive Traffic",
  "tags": ["crawler"],
  "is_malicious": false,
  "req_count": 15,
  "req_rate": 0.25,
  "is_spike": false,
  "first_seen_iso": "2025-03-30T12:00:00",
  "last_seen_iso": "2025-03-30T12:01:00",
  "path_summary": ["/", "/robots.txt"]
}
```

### `notable` (array)

Organizations identified by rDNS or ASN:

```json
{
  "label": "ASN: AS16509 (Amazon.com, Inc.) | Confidence: hosting provider only | Actor: unknown",
  "ips": ["203.0.113.10", "203.0.113.11"],
  "count": 2
}
```

Label prefixes determine categorization:
- `RDNS:` -- reverse DNS identified
- `ASN:` -- ASN-based identification
- `Verified Actor:` -- confirmed bot identity (filtered from display)

### `summary` (object)

```json
{
  "total_requests": 6324,
  "unique_origin_ips": 630,
  "updated_at": "2025-03-30T21:37:55",
  "countries": {
    "US": { "legit": 136, "bots": 88, "malicious": 61, "name": "United States" }
  }
}
```

## Configuration

All configuration is in `app.js` constants at the top of the file:

| Constant | Purpose |
|----------|---------|
| `BLOCKED_COUNTRIES` | ISO country codes shown as blocked on the map |
| `CLOUD_PROVIDERS` | ASN-to-provider mapping for grouping notable orgs |
| `IGNORED_RDNS` | rDNS domains to exclude from notables (scanners) |
| `MALICIOUS_PATHS` | Regex patterns that flag sessions as malicious |
| `TRAFFIC_COLORS` | Color scheme for traffic type categories |

## Traffic Types

| Type | Color | Criteria |
|------|-------|----------|
| Legitimate | Cyan `#00f2ff` | No bot/cloud/hosting/malicious flags |
| Cloud | Purple `#a855f7` | `geo.is_cloud` is true |
| Hosting | Green `#22c55e` | `geo.is_hosting` is true |
| Bot | Amber `#ffaa00` | `geo.is_bot` is true |
| Malicious | Red `#f43f5e` | `is_malicious` or hits scanner path patterns |

## Generating data.json

The dashboard is data-source agnostic. You can generate `data.json` from any web server log format. A typical pipeline:

1. Parse nginx/Apache access logs
2. Enrich IPs with GeoIP and ASN data (MaxMind, ipinfo.io, etc.)
3. Cluster requests by IP into sessions
4. Classify intent (bot detection, rate analysis)
5. Output the JSON structure above

## Dependencies

All loaded from CDN (no `npm install` required):

- [Leaflet](https://leafletjs.com/) v1.9.4 -- Map rendering
- [Chart.js](https://www.chartjs.org/) -- Status/type/volume charts
- [chartjs-plugin-datalabels](https://chartjs-plugin-datalabels.netlify.app/) -- Chart label overlays
- [Lucide Icons](https://lucide.dev/) -- UI icons
- [Inter](https://rsms.me/inter/) -- Font

## License

MIT
