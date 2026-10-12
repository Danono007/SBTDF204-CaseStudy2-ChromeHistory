# SBT-DF204 Case Study 2: Chrome Web History Analysis Toolkit

Reusable SQL and shell tooling for examining a Google Chrome `History` SQLite database in a computer forensics workflow. Built for **SBT-DF204 Computer Forensics** coursework (case study: reconstructing suspected activity from Chrome web history).

> **Academic note.** This is student coursework on a training dataset. The repository contains **no evidence files, no case findings and no report**. It only contains generic, read-only tooling. Do not copy it as submitted work for your own assignment.

## Contents

| Path | Description |
|------|-------------|
| `README.md` | This file |
| `scripts/collect_evidence.sh` | Runs standard read-only queries and saves each result to `output/` |
| `.gitignore` | Keeps evidence, screenshots, reports and outputs out of git |

## What the script does

| Section | Purpose |
|---------|---------|
| 1 | Hashes of the original and working copy, table list, schema (`PRAGMA table_info`), row counts |
| 2 | WebKit-to-UTC timestamp conversion check |
| 3 | Full visit timeline, plus `full_timeline.csv` |
| 4 | Transition decoding, `from_visit` chains, `opener_visit`, search terms |
| 5 | Example site-specific queries (classified ads, webmail) |
| 6 | Optional string search (`TXID`), download records and redirect chains, blockchain and exchange sites |
| 7 | Sanity checks (integrity check, counts, first and last visit time) |

## Evidence handling (recommended workflow)

1. Hash the original: `sha256sum History`, then `chmod 444 History`.
2. Work only on a copy (`working/History_working`).
3. Open it read-only (`sqlite3 -readonly`, `sqlitebrowser -R`). The script always uses `-readonly`.
4. Re-hash the original at the end and confirm it is unchanged.

## Requirements

- Linux (developed on Kali)
- `bash`, `sqlite3`, `sha256sum`
- Optional: DB Browser for SQLite

## Usage

```bash
mkdir -p original working output
cp /path/to/History original/History
sha256sum original/History
chmod 444 original/History
cp original/History working/History_working

chmod +x scripts/collect_evidence.sh
./scripts/collect_evidence.sh            # all sections
./scripts/collect_evidence.sh 3          # one section (1 to 7)

# optional settings
EXPECTED_SHA256=<your recorded hash> ./scripts/collect_evidence.sh 1
TXID=<string to search for> ./scripts/collect_evidence.sh 6
DB=path/to/copy OUT=path/to/results ./scripts/collect_evidence.sh
```

## Method notes

- Chrome timestamps are microseconds since 1601-01-01 UTC. Convert with `datetime(t/1000000 + strftime('%s','1601-01-01'),'unixepoch')`.
- `transition & 255` gives the core type (0 link, 5 typed/omnibox, 7 form submit). Higher bits are qualifiers (chain start/end, redirects, address bar). Use `printf('%08X', transition & 4294967295)`, because DB Browser may display signed values.
- `urls.title` stores only the latest title per URL, so earlier visit titles may differ.
- Browser artefacts show activity on a profile. They do not by themselves prove who performed it.

## Safety

Never visit live URLs, wallet addresses or transactions found in evidence. Analyse them as text only.


## Licence

MIT. See `LICENSE`.

