# keypadzoom-sites

Remote site pack repository for the [KeypadZoom](https://github.com/deanmauriceellis-cloud/keypadzoom) Chrome extension.

This repo hosts the JSON data files that map KeypadZoom's 4,558 navigation endpoints to real URLs. The extension checks this CDN once every 24 hours for updates — allowing URL corrections and additions without publishing a new extension version.

## How It Works

```
User navigates in extension
        ↓
SiteResolver checks chrome.storage.local (Layer 3 — cached remote)
        ↓ miss
Falls back to bundled JSON (Layer 2 — ships with extension)
        ↓
Background: 24hr non-blocking CDN check
        ↓ new version detected
Differential fetch (only changed key files, by checksum)
        ↓
Updated data cached in chrome.storage.local
```

## Directory Structure

```
keypadzoom-sites/
├── sites-latest/          ← Live site packs (served via GitHub Pages)
│   ├── manifest.json      ← Version, checksums, key metadata
│   ├── brands.json        ← Domain → brand code + color mapping
│   ├── key1-social.json   ← Social Media (568 endpoints)
│   ├── key2-health.json   ← Health & Wellness (540 endpoints)
│   ├── key3-games.json    ← Games (568 endpoints)
│   ├── key4-technology.json ← Technology (701 endpoints)
│   ├── key6-entertainment.json ← Entertainment (554 endpoints)
│   ├── key7-news.json     ← News (533 endpoints)
│   ├── key8-money.json    ← Money & Career (568 endpoints)
│   └── key9-shopping.json ← Shopping (526 endpoints)
├── schemas/               ← JSON validation schemas
│   ├── site-pack.schema.json
│   ├── manifest.schema.json
│   └── brands.schema.json
└── .github/workflows/     ← CI: validates on PR, logs deployment on merge
```

## Site Pack Format

Each `keyX-*.json` file maps tree paths to arrays of site entries:

```json
{
  "key": 4,
  "label": "Technology",
  "sites": {
    "1.1.1": [
      {
        "d": "chatgpt.com",
        "n": "ChatGPT",
        "u": "https://chatgpt.com"
      }
    ],
    "1.1.2": [
      {
        "d": "claude.ai",
        "n": "Claude",
        "u": "https://claude.ai"
      }
    ]
  }
}
```

**Fields:**
| Field | Required | Description |
|-------|----------|-------------|
| `d` | Yes | Domain name (used for brand resolution) |
| `n` | Yes | Display name shown on the launchpad tile |
| `u` | Yes | Full URL to open (must be `https://` or `http://`) |
| `b` | No | 2-letter brand code override (normally resolved from `brands.json`) |
| `c` | No | Hex brand color override (normally resolved from `brands.json`) |

## How to Update URLs

### Quick edit (single endpoint)

1. Open the relevant `keyX-*.json` file
2. Find the tree path (e.g., `"1.1.1"`)
3. Update the `d`, `n`, and/or `u` fields
4. Update the checksum in `manifest.json` (run `sha256sum keyX-*.json | cut -c1-8`)
5. Bump the version in `manifest.json` (increment the last digit)
6. Commit and push

### Batch update

1. Edit the site pack files as needed
2. Run the checksum update:
   ```bash
   for f in sites-latest/key*.json; do
     echo "$(basename $f): $(sha256sum $f | cut -c1-8)"
   done
   echo "brands: $(sha256sum sites-latest/brands.json | cut -c1-8)"
   ```
3. Update all checksums in `manifest.json`
4. Bump version and `updated` timestamp
5. Commit and push

### Adding a new brand

1. Add the domain entry to `brands.json`:
   ```json
   "newsite.com": { "b": "NS", "c": "#FF5722" }
   ```
2. Ensure the 2-letter code is unique (CI will catch duplicates)
3. Update `brandsChecksum` in `manifest.json`

## Validation

The GitHub Actions workflow validates every push and PR:

- **Schema validation** — all JSON files conform to their schemas
- **Checksum verification** — manifest checksums match actual file hashes
- **Brand code uniqueness** — no duplicate 2-letter codes
- **Endpoint count verification** — manifest counts match actual site entries

## CDN URL

```
https://deanmauriceellis-cloud.github.io/keypadzoom-sites/sites-latest/
```

The extension fetches `manifest.json` from this base URL, then selectively fetches only the key files whose checksums have changed.
