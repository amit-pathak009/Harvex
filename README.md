# 🕸️ Harvex — Ultimate URL Harvester 2026

> **The most comprehensive, Tor-routed, multi-tool URL harvesting framework for authorized security reconnaissance.**

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Kali-informational)](https://kali.org)
[![Tor](https://img.shields.io/badge/Tor-Enabled-purple?logo=tor-project)](https://torproject.org)
[![Tools](https://img.shields.io/badge/Tools-10%2B%20Integrated-orange)](#integrated-tools)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen)]()
![Visitors](https://visitor-badge.laobi.icu/badge?page_id=amit-pathak009.harvex)

---

## 📖 Table of Contents

- [What is Harvex?](#what-is-harvex)
- [Key Features](#key-features)
- [Integrated Tools](#integrated-tools)
- [How It Works](#how-it-works)
- [Installation](#installation)
- [Usage](#usage)
- [CLI Arguments](#cli-arguments)
- [Tor & Anonymity](#tor--anonymity)
- [TheTimeMachine Integration](#thetimemachine-integration)
- [Output & Post-Processing](#output--post-processing)
- [Dry Run & Tool Check](#dry-run--tool-check)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)
- [Legal Disclaimer](#legal-disclaimer)
- [Credits](#credits)

---

## What is Harvex?

**Harvex** is a Python-based URL harvesting framework designed for **authorized penetration testing, bug bounty reconnaissance, and security research**. It orchestrates 10+ industry-standard tools simultaneously — passive archive crawlers, active web spiders, and JavaScript endpoint extractors — all routed through **Tor with automatic IP rotation** between each tool execution.

Unlike running individual tools manually, Harvex:
- Aggregates results from all tools into a **single deduplicated output**
- **Rotates your Tor exit IP** before each tool so no single IP hammers the target
- Streams live progress with a **real-time URL counter and elapsed timer**
- Displays your **Tor circuit's entry and exit node fingerprints** per run
- Automatically deduplicates and optionally filters for **parameterized URLs only**

**Use case:** You're doing a bug bounty on `target.com`. Instead of running waymore, gau, katana, gospider, etc. one by one and manually combining results, Harvex runs everything in sequence, rotates Tor between each tool, deduplicates with `uro`, and hands you a clean URL list ready for fuzzing or scanning.

---

## Key Features

### Tor Integration with IP Rotation
- Connects to Tor's **ControlPort 9051** via `stem`
- Sends `NEWNYM` signal before each tool to get a **fresh exit node**
- Displays live countdown during circuit rotation
- Shows **Exit IP**, **Entry Node fingerprint**, and **Exit Node fingerprint** in every tool header
- Falls back gracefully if Tor is slow or unavailable

### Real-Time Live Progress
- Each tool shows a **continuously ticking timer** — updated every second via a background thread, never freezes even when the tool is idle between network requests
- Live URL counter updates as URLs are discovered
- Clean overwriting status line — no scrolling spam, no duplicate lines

### 10+ Tool Orchestration
Passive (archive) and active (spider) tools run in a logical sequence:
1. **Archive/passive** tools first (waymore, gau, waybackurls, urlfinder)
2. **Active crawlers** second (katana, hakrawler, cariddi, gospider, URLFinder, TheTimeMachine)

### Domain-Scoped Filtering
- Strict regex ensures **only URLs belonging to the target domain** (and its subdomains) are collected
- Prevents pollution from third-party CDN/analytics URLs that tools often emit
- Supports both `evil.com` and `www.evil.com`, `api.evil.com`, etc.

### Multi-Domain / List Mode
- Pass a single domain with `-t` or a **file of domains** with `-l`
- List-optimized commands are used automatically (e.g., `cat list | waybackurls` instead of looping)

### Post-Processing Pipeline
- **URO deduplication** — removes duplicate parameterized URLs intelligently
- **`--params-only`** — filter output to URLs containing query parameters (ideal for fuzzing)
- **`--live`** — probe all collected URLs with `httpx` and keep only live ones
- **`--include-subs`** — enumerate subdomains first via `subfinder`, then harvest all of them

### Quiet / Pipe-Friendly Mode
- `--quiet` suppresses all UI output and prints only the final output file path
- Ideal for use in shell pipelines or automated workflows

---

## Integrated Tools

| Tool | Type | Source | Proxy Support |
|------|------|--------|---------------|
| **waymore** | Passive archive | `pip3 install waymore` | ✅ proxychains4 |
| **gau** | Passive archive | Go install | ✅ `--proxy` flag |
| **waybackurls** | Passive archive | Go install | ❌ direct (no proxy support) |
| **urlfinder** | Passive archive | Go install | ✅ `-proxy` flag |
| **katana** | Active crawler + JS | Go install | ✅ `-proxy` flag |
| **hakrawler** | Active crawler | Go install | ❌ direct |
| **cariddi** | Active crawler | Go install | ✅ `-proxy` flag |
| **gospider** | Active spider | Go install | ✅ `-p` flag |
| **URLFinder** (pingc0y) | Active JS parser | Manual build | ✅ `-x` flag |
| **TheTimeMachine** | Wayback fetcher | Manual clone | ✅ proxychains4 |
| **subfinder** | Subdomain enum | Go install | ✅ `-proxy` flag |
| **httpx** | Live URL prober | Go install | ✅ `-proxy` flag |
| **uro** | URL deduplication | `pip3 install uro` | N/A |

---

## How It Works

```
Target Domain(s)
       │
       ▼
 Tor Circuit Init  ←──── stem ControlPort 9051
       │
       ▼
 ┌─────────────────────────────────┐
 │  For each tool:                 │
 │  1. NEWNYM → rotate Tor IP      │
 │  2. Fetch circuit fingerprints  │
 │  3. Execute tool via proxy      │
 │  4. Stream stdout → filter URLs │
 │  5. Add to deduplicated pool    │
 └─────────────────────────────────┘
       │
       ▼
  URO Deduplication (optional)
       │
       ▼
  Params Filter / Live Filter (optional)
       │
       ▼
  results/harvested_urls_YYYYMMDD_HHMMSS.txt
```

Each tool's output is **streamed line by line** and matched against a domain-scoped regex in real time, so URLs are collected and counted as the tool runs — not after it finishes.

---

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/amit-pathak009/harvex.git
cd harvex
```

### 2. Install Python dependencies

```bash
pip3 install -r requirements.txt --break-system-packages
```

### 3. Install Go tools

```bash
go install -v github.com/lc/gau/v2/cmd/gau@latest
go install -v github.com/tomnomnom/waybackurls@latest
go install -v github.com/projectdiscovery/urlfinder/cmd/urlfinder@latest
go install -v github.com/projectdiscovery/katana/cmd/katana@latest
go install -v github.com/hakluke/hakrawler@latest
go install -v github.com/edoardottt/cariddi/cmd/cariddi@latest
go install -v github.com/jaeles-project/gospider@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# Make Go binaries available system-wide
sudo cp $HOME/go/bin/* /usr/local/bin
```

### 4. Install URLFinder (manual build)

```bash
git clone https://github.com/pingc0y/URLFinder
cd URLFinder
go build -o URLFinder main.go
sudo cp URLFinder /usr/local/bin/
cd ..
```

### 5. Install TheTimeMachine (optional)

```bash
git clone https://github.com/anmolksachan/TheTimeMachine /home/kali/tools/TheTimeMachine
```

Then set the path at the top of `harvex.py`:

```python
THETIMEMACHINE_PATH = "/home/kali/tools/TheTimeMachine/thetimemachine.py"
```

### 6. Install Tor + proxychains4

```bash
sudo apt install tor proxychains4 -y
sudo usermod -aG debian-tor $USER
newgrp debian-tor

# Start and enable Tor
sudo systemctl enable tor
sudo systemctl start tor
```

**Recommended `/etc/tor/torrc` additions** for faster IP rotation:

```
MaxCircuitDirtiness 10
NewCircuitPeriod 10
```

### 7. Verify everything is installed

```bash
python3 harvex.py --dry-run
```

---

## Usage

### Basic — single domain, Tor enabled

```bash
python3 harvex.py -t target.com
```

### Single domain, no Tor (faster, not anonymous)

```bash
python3 harvex.py -t target.com --no-proxychains
```

### List of domains

```bash
python3 harvex.py -l domains.txt
```

### With subdomain enumeration + live filtering

```bash
python3 harvex.py -t target.com --include-subs --live
```

### Parameters only (for fuzzing)

```bash
python3 harvex.py -t target.com --params-only
```

### Skip specific tools

```bash
python3 harvex.py -t target.com --skip katana,gospider
```

### Quiet mode (pipe output to another tool)

```bash
python3 harvex.py -t target.com --quiet | xargs -I{} cat {}
```

### Save raw tool outputs for debugging

```bash
python3 harvex.py -t target.com --save-raw
```

---

## CLI Arguments

| Argument | Short | Description |
|----------|-------|-------------|
| `--target DOMAIN` | `-t` | Single target domain (e.g. `example.com`) |
| `--list FILE` | `-l` | File containing one domain per line |
| `--output FILE` | `-o` | Custom output filename |
| `--output-dir DIR` | | Output directory (default: `results/`) |
| `--include-subs` | | Enumerate subdomains with subfinder before harvesting |
| `--live` | | Filter results through httpx — keep only live URLs |
| `--no-uro` | | Skip uro deduplication step |
| `--params-only` | | Keep only URLs that contain query parameters |
| `--save-raw` | | Save each tool's raw output to `results/raw/` |
| `--skip TOOLS` | | Comma-separated list of tool names to skip |
| `--no-proxychains` | | Disable Tor/proxychains4 (run direct) |
| `--dry-run` | | Check which tools are installed, then exit |
| `--auto-install` | | Print all install commands, then exit |
| `--quiet` | `-q` | Suppress all UI, print only output file path |

---

## Tor & Anonymity

Harvex uses a **two-layer Tor integration**:

**Layer 1 — proxychains4** wraps Python tools (`waymore`, `TheTimeMachine`) at the process level, forcing all their connections through Tor's SOCKS5 proxy at `127.0.0.1:9050`.

**Layer 2 — native proxy flags** are used for Go tools that support them natively (`--proxy`, `-proxy`, `-p`, `-x`). This is more reliable than proxychains for Go tools.

**IP Rotation** happens automatically before each tool:
1. Harvex calls `NEWNYM` via stem's ControlPort
2. Waits out the `NewCircuitDirtiness` timer (shown as a live countdown)
3. Verifies the new exit IP via `api.ipify.org` through the proxy
4. Fetches the entry and exit node fingerprints from the active circuit
5. Displays all of this in the tool header panel

**Note:** `waybackurls` and `hakrawler` do not support proxy flags and run direct. Their requests go to public APIs (Wayback Machine) which is generally acceptable.

---

## TheTimeMachine Integration

TheTimeMachine is handled differently from other tools because it **writes results to disk** rather than printing them to stdout.

Harvex uses a dedicated runner (`_run_thetimemachine`) that:
1. Rotates Tor and shows the circuit panel
2. Executes the script via proxychains4 with `cd` into the script's own directory (required for its relative imports)
3. Shows a live ticking timer while it runs
4. After completion, scans the `content/<domain>/` output directory for `.txt` files
5. Parses all discovered files and imports matching URLs into the main pool

The script path is configured at the top of `harvex.py`:

```python
THETIMEMACHINE_PATH = "/home/kali/tools/TheTimeMachine/thetimemachine.py"
```

---

## Output & Post-Processing

All results are saved to the `results/` directory by default.

```
results/
├── harvested_urls_20260301_143022.txt   ← final deduplicated URL list
└── raw/                                  ← per-tool raw output (--save-raw only)
    ├── waymore_target.com.txt
    ├── gau_target.com.txt
    └── ...
```

The final summary table shows:
- Per-tool URL counts and percentage share
- Total raw URLs collected across all tools
- Total final URLs after deduplication
- Elapsed time and Tor status

---

## Dry Run & Tool Check

Run `--dry-run` at any time to audit your installation without performing any harvest:

```bash
python3 harvex.py --dry-run
```

Output:

```
╔══════════════════════════╦══════════════════╦══════════════════════════════════╗
║  🔧  Tool                ║  ✅  Status      ║  📦  Install                     ║
╠══════════════════════════╬══════════════════╬══════════════════════════════════╣
║  waymore                 ║  ✅ Installed    ║  —                               ║
║  gau                     ║  ✅ Installed    ║  —                               ║
║  katana                  ║  ❌ Missing      ║  go install ...katana@latest     ║
╚══════════════════════════╩══════════════════╩══════════════════════════════════╝
```

---

## Examples

### Bug Bounty Recon

```bash
# Full recon with subs, live filter, params only
python3 harvex.py -t hackerone.com --include-subs --live --params-only -o hackerone_params.txt
```

### Fast passive-only (skip active crawlers)

```bash
python3 harvex.py -t target.com --skip katana,hakrawler,cariddi,gospider,URLFinder
```

### Test your setup on a safe target

```bash
python3 harvex.py -t testphp.vulnweb.com --no-proxychains
```

### Pipe results directly into nuclei

```bash
python3 harvex.py -t target.com --quiet --live | xargs -I{} nuclei -l {}
```

---

## Troubleshooting

**`Tor controller error: Authentication failed`**
```bash
sudo usermod -aG debian-tor $USER
newgrp debian-tor
# Or restart your session
```

**`proxychains4 not found`**
```bash
sudo apt install proxychains4 -y
```

**`Entry/Exit nodes showing —`**
This means Tor has circuits but stem can't read their paths. Check that your user is in the `debian-tor` group and that `CookieAuthentication 1` is set in `/etc/tor/torrc`.

**`TheTimeMachine: 0 URLs`**
- Verify `THETIMEMACHINE_PATH` is set correctly at the top of `harvex.py`
- Run `python3 /path/to/thetimemachine.py target.com --fetch` manually to confirm it works
- Check the stderr snippet printed after a 0-URL result

**Go tools not found after install**
```bash
sudo cp $HOME/go/bin/* /usr/local/bin
```

**`waymore` returning 0 URLs**
waymore has its own config file at `~/.config/waymore/config.yml`. Some sources require API keys. Check `waymore --help` for source configuration.

---

## Legal Disclaimer

> **Harvex is intended for authorized security testing only.**
>
> You must have explicit written permission from the target organization before running this tool against any system. Unauthorized use against systems you do not own or have permission to test is **illegal** in most jurisdictions and may result in criminal prosecution.
>
> The authors assume no liability for misuse. Use responsibly, ethically, and legally.

---

## Credits

Harvex is a framework that orchestrates existing open-source tools. All credit for the underlying capabilities goes to their respective authors:

- [waymore](https://github.com/xnl-h4ck3r/waymore) — xnl-h4ck3r
- [gau](https://github.com/lc/gau) — lc
- [waybackurls](https://github.com/tomnomnom/waybackurls) — tomnomnom
- [urlfinder](https://github.com/projectdiscovery/urlfinder) — ProjectDiscovery
- [katana](https://github.com/projectdiscovery/katana) — ProjectDiscovery
- [hakrawler](https://github.com/hakluke/hakrawler) — hakluke
- [cariddi](https://github.com/edoardottt/cariddi) — edoardottt
- [gospider](https://github.com/jaeles-project/gospider) — jaeles-project
- [URLFinder](https://github.com/pingc0y/URLFinder) — pingc0y
- [TheTimeMachine](https://github.com/anmolksachan/TheTimeMachine) — anmolksachan
- [subfinder](https://github.com/projectdiscovery/subfinder) — ProjectDiscovery
- [httpx](https://github.com/projectdiscovery/httpx) — ProjectDiscovery
- [uro](https://github.com/s0md3v/uro) — s0md3v
- [stem](https://stem.torproject.org/) — Tor Project
