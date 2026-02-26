#  Harvex — Ultimate URL Harvester

> A fast, multi-tool URL harvesting framework for bug bounty hunters and security researchers. Harvex chains together the best open-source recon tools to collect, deduplicate, and filter URLs from a single target or a list of domains.

---

## ✨ Features

- **Multi-tool chaining** — runs `waymore`, `gau`, `waybackurls`, `urlfinder`, `katana`, `hakrawler`, `cariddi`, and `gospider` in parallel
- **Live URL count** — real-time progress display while harvesting
- **Smart deduplication** — optional `uro` post-processing to remove redundant parameterized URLs
- **Live filtering** — optional `httpx` integration to keep only live endpoints
- **Subdomain expansion** — integrates `subfinder` to discover and harvest across subdomains
- **Proxy support** — pass any HTTP or SOCKS5 proxy to supported tools
- **JS endpoint extraction** — crawl JavaScript files for hidden endpoints via `katana`
- **Flexible output** — save results as plain TXT; JSON and HTML report flags are available
- **Per-tool contribution summary** — see exactly how many URLs each tool found
- **Skip individual tools** — bypass tools you don't have or don't need
- **Save raw output** — keep raw tool outputs for manual review

---

## 📦 Requirements

### Python

```
Python >= 3.8
rich
```

Install Python dependencies:

```bash
pip install -r requirements.txt
```

### External Tools (install separately)

Harvex calls these CLI tools — install whichever you need:

| Tool | Install |
|------|---------|
| [waymore](https://github.com/xnl-h4ck3r/waymore) | `pip install waymore` |
| [gau](https://github.com/lc/gau) | `go install github.com/lc/gau/v2/cmd/gau@latest` |
| [waybackurls](https://github.com/tomnomnom/waybackurls) | `go install github.com/tomnomnom/waybackurls@latest` |
| [urlfinder](https://github.com/projectdiscovery/urlfinder) | `go install github.com/projectdiscovery/urlfinder/cmd/urlfinder@latest` |
| [katana](https://github.com/projectdiscovery/katana) | `go install github.com/projectdiscovery/katana/cmd/katana@latest` |
| [hakrawler](https://github.com/hakluke/hakrawler) | `go install github.com/hakluke/hakrawler@latest` |
| [cariddi](https://github.com/edoardottt/cariddi) | `go install github.com/edoardottt/cariddi/cmd/cariddi@latest` |
| [gospider](https://github.com/jaeles-project/gospider) | `go install github.com/jaeles-project/gospider@latest` |
| [uro](https://github.com/s0md3v/uro) *(optional)* | `pip install uro` |
| [httpx](https://github.com/projectdiscovery/httpx) *(optional)* | `go install github.com/projectdiscovery/httpx/cmd/httpx@latest` |
| [subfinder](https://github.com/projectdiscovery/subfinder) *(optional)* | `go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest` |
| [URLFinder pingc0y](https://github.com/pingc0y/URLFinder/) | `git clone https://github.com/pingc0y/URLFinder/` |


> Harvex gracefully skips tools that are not installed. You don't need all of them to get started.

---

## 🚀 Installation

```bash
git clone https://github.com/yourusername/harvex.git
cd harvex
pip install -r requirements.txt
chmod +x harvex.py
```

Optionally add to your PATH:

```bash
sudo ln -s $(pwd)/harvex.py /usr/local/bin/harvex
```

---

## 🛠️ Usage

```
usage: harvex.py [-h] [-t TARGET] [-l LIST] [-p PROXY] [-o OUTPUT]
                 [-oj OUTPUT_JSON] [-oh OUTPUT_HTML] [--output-dir OUTPUT_DIR]
                 [--include-subs] [--live] [--no-uro] [--params-only]
                 [--js-endpoints] [--save-raw] [--depth DEPTH] [--js]
                 [--skip SKIP] [--timeout TIMEOUT]
```

### Options

| Flag | Description |
|------|-------------|
| `-t, --target` | Single domain to harvest (e.g. `example.com`) |
| `-l, --list` | Path to a file containing one domain per line |
| `-p, --proxy` | Proxy URL (e.g. `http://127.0.0.1:8080` or `socks5://127.0.0.1:1080`) |
| `-o, --output` | Output filename for the final TXT file |
| `-oj, --output-json` | Output filename for JSON format |
| `-oh, --output-html` | Output filename for HTML report |
| `--output-dir` | Directory to save all output files (default: `results/`) |
| `--include-subs` | Also enumerate and harvest subdomains via `subfinder` |
| `--live` | Filter final URL list to live endpoints only (requires `httpx`) |
| `--no-uro` | Skip `uro` deduplication step |
| `--params-only` | Only keep URLs that contain query parameters |
| `--js-endpoints` | Extract endpoints from JS files |
| `--js` | Enable JS crawling in `katana` (`-jc` flag) |
| `--save-raw` | Save raw output from each tool to `results/raw/` |
| `--depth` | Crawl depth for crawlers (default: `10`) |
| `--skip` | Comma-separated list of tools to skip (e.g. `waymore,cariddi`) |
| `--timeout` | Per-tool timeout in seconds (default: `1800`) |

---

## 📋 Examples

**Harvest a single domain:**
```bash
python harvex.py -t example.com
```

**Harvest a list of domains and save to a file:**
```bash
python harvex.py -l domains.txt -o output.txt
```

**Include subdomains + live filter + params only:**
```bash
python harvex.py -t example.com --include-subs --live --params-only
```

**Use a proxy and enable JS crawling:**
```bash
python harvex.py -t example.com -p http://127.0.0.1:8080 --js
```

**Skip slow tools and save raw outputs:**
```bash
python harvex.py -t example.com --skip waymore,cariddi --save-raw
```

**Set a custom output directory:**
```bash
python harvex.py -l domains.txt --output-dir /tmp/recon_results
```

---

## 📁 Output Structure

```
results/
├── harvested_urls_20260101_120000.txt   # Final clean URL list
└── raw/                                 # Raw tool outputs (with --save-raw)
    ├── gau_example_com.txt
    ├── katana_example_com.txt
    └── ...
```

---

## 🧠 How It Works

1. Harvex takes one or more domains as input
2. For each domain, it runs all configured tools **in parallel** (up to 5 workers)
3. URLs are extracted from tool output using regex, then deduplicated
4. Optionally, `uro` removes redundant parameterized URLs
5. Optionally, `httpx` filters down to live endpoints only
6. Final results are saved to the output directory with a timestamp

---

## ⚠️ Disclaimer

Harvex is intended for **authorized security testing and bug bounty programs only**. Do not use this tool against systems you do not have explicit permission to test. The author is not responsible for any misuse.

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.
