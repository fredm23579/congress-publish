# Congress Publisher

A Python 3 CLI tool that downloads U.S. Congressional bill and amendment data
from official government APIs and publishes it to a GitHub repository.
Supports manual runs and automated self-updating via built-in scheduling.

---

## Features

- Fetches bill and amendment data from the **Congress.gov API v3** (official
  Library of Congress API)
- Publishes data to the **`gh-pages`** branch of any GitHub repository
- **Self-updating scheduler**: run once or keep running on `hourly`, `daily`,
  `weekly`, or custom (`6h`, `30m`, …) intervals
- Writes structured JSON output — a summary file plus one file per bill
- Token-based GitHub authentication (no passwords)
- `--dry-run` mode for safe previewing
- `--verbose` mode for debugging
- Automatic current-congress detection (no manual number lookup needed)

---

## Quick Start

### 1. Install dependencies

```bash
python3 -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure

```bash
cp config.yml.example config.yml
```

Edit `config.yml`:

```yaml
github:
  token: ghp_YOUR_PERSONAL_ACCESS_TOKEN   # repo scope required
  repo: git@github.com:YOU/congress-data.git

repo: /path/to/local/congress/data

congress_api_key: YOUR_FREE_API_KEY   # https://api.congress.gov/sign-up/
```

### 3. Run

```bash
# Update the current congress (119th as of 2025–2026)
./publish --current

# Preview first (no writes, no pushes)
./publish --current --dry-run
```

---

## Usage

```
usage: publish [--current | --congresses N[,N…]] [options]

options:
  --current             update the current congress automatically
  --congresses N[,N…]   comma-separated congress numbers (e.g. 117,118,119)
  --config FILE         path to YAML config (default: config.yml)
  --dry-run             preview without writing or pushing anything
  --no-amendments       fetch bills only; skip amendment data
  --schedule INTERVAL   keep running on a schedule (see below)
  --verbose, -v         enable debug-level logging
```

### Examples

```bash
# One-shot updates
./publish --current
./publish --congresses=119
./publish --congresses=117,118,119

# Bills only
./publish --current --no-amendments

# Safe preview
./publish --current --dry-run --verbose

# Self-updating
./publish --current --schedule=daily      # once per day
./publish --current --schedule=6h         # every 6 hours
./publish --current --schedule=30m        # every 30 minutes
./publish --current --schedule=weekly     # once per week
./publish --current --schedule=hourly     # every hour
```

---

## Configuration Reference

`config.yml` (copy from `config.yml.example`):

| Field | Required | Description |
|---|---|---|
| `github.token` | Recommended | GitHub Personal Access Token (`repo` scope). Create at [github.com/settings/tokens](https://github.com/settings/tokens). SSH key auth also works — leave blank and configure SSH. |
| `github.repo` | Yes | Target repository URL. SSH: `git@github.com:USER/REPO.git`  HTTPS: `https://github.com/USER/REPO.git` |
| `repo` | Yes | Local directory where data is written before publishing |
| `congress_api_key` | Recommended | Free key from [api.congress.gov/sign-up](https://api.congress.gov/sign-up/). Without it requests are rate-limited. |
| `govinfo_api_key` | Optional | Free key from [api.data.gov/signup](https://api.data.gov/signup/) |

> `config.yml` is git-ignored. Never commit it — it contains credentials.

---

## Data Sources

| Source | URL | Notes |
|---|---|---|
| Congress.gov API v3 | https://api.congress.gov/v3 | Primary; official Library of Congress API |
| GovInfo API | https://api.govinfo.gov | Supplemental; Government Publishing Office |

Both APIs are **free** and operated by the U.S. federal government. An API key
is optional but strongly recommended to avoid rate limits.

---

## Published Output Format

After a successful run the local `repo` directory contains:

```
<repo>/
└── 119/                      ← congress number
    ├── metadata.json          ← summary: counts, fetch time, source
    ├── bills.json             ← full paginated bill list
    ├── amendments.json        ← full paginated amendment list
    └── bills/
        ├── hr1.json           ← individual bill files
        ├── s42.json
        └── …
```

Everything is committed to the `gh-pages` branch and pushed to GitHub.

---

## Self-Update / Automation

Pass `--schedule` to keep the process running and re-fetch automatically:

```bash
# Background daemon (nohup)
nohup ./publish --current --schedule=daily > congress-publish.log 2>&1 &

# systemd unit (recommended for servers)
# See docs/systemd-example.service for a template
```

Schedule format options: `hourly`, `daily`, `weekly`, `Xh` (hours), `Xm` (minutes).

---

## Current Congress

The current U.S. Congress is calculated automatically:

```
congress = (year - 1789) // 2 + 1   (year rounded to nearest odd year)
```

| Years | Congress |
|---|---|
| 2021–2022 | 117th |
| 2023–2024 | 118th |
| 2025–2026 | **119th** ← current |
| 2027–2028 | 120th |

---

## Requirements

- Python 3.8+
- `GitPython >= 3.1.41`
- `PyYAML >= 6.0.1`
- `requests >= 2.32.3`
- `schedule >= 1.2.2`

---

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with `--dry-run --verbose`
5. Open a pull request

---

## License

Public domain — see the [unitedstates project](https://github.com/unitedstates).
