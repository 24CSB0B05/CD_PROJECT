[README.md](https://github.com/user-attachments/files/26332795/README.md)
# CVE Vulnerability Description Parser

> **A compiler-inspired NLP tool that parses CVE descriptions, maps them to known vulnerability patterns, and enriches each finding with CWE IDs, mitigations, and code-construct linkage.**

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture & Pipeline](#architecture--pipeline)
3. [Project Structure](#project-structure)
4. [Setup & Installation](#setup--installation)
5. [Usage](#usage)
6. [Module Reference](#module-reference)
7. [Output Format](#output-format)
8. [Supported Vulnerability Types](#supported-vulnerability-types)
9. [Sample Run](#sample-run)
10. [Extending the Tool](#extending-the-tool)
11. [Compiler Concepts Applied](#compiler-concepts-applied)

---

## Project Overview

Security teams deal with hundreds of CVE descriptions daily. Reading each one manually to determine the vulnerability class, root cause, affected component, severity, and appropriate mitigation is slow and error-prone.

This tool applies **compiler design principles** — lexical analysis, parsing, semantic analysis, and IR generation — to automate that workflow:

- Parse raw CVE text using NLP tokenisation
- Pattern-match to 11 known vulnerability categories
- Semantically extract root cause, impact, and affected component
- Enrich records with CWE IDs, mitigations, and linked code constructs
- Output structured JSON + a formatted terminal report

---

## Architecture & Pipeline

```
 Raw CVE Text
      │
      ▼
 ┌─────────────┐
 │   LEXER     │  Lowercase → normalise punctuation → strip stop-words → tokens[]
 └──────┬──────┘
        │
        ▼
 ┌─────────────┐
 │   PARSER    │  Keyword pattern-matching with weighted scoring → vulnerability_type
 └──────┬──────┘
        │
        ▼
 ┌──────────────────┐
 │ SEMANTIC ANALYSIS│  Rule-based extraction → root_cause, impact, component, severity
 └──────┬───────────┘
        │
        ▼
 ┌─────────────┐
 │ IR BUILDER  │  Assembles full record + CWE enrichment + code-construct linkage
 └──────┬──────┘
        │
        ▼
 ┌─────────────────────────────────┐
 │  OUTPUT                         │
 │  • output.json  (structured IR) │
 │  • Terminal report + statistics │
 └─────────────────────────────────┘
```

---

## Project Structure

```
cve_parser/
│
├── main.py          # Entry point — orchestrates the full pipeline
├── lexer.py         # Tokenisation and text normalisation
├── parser.py        # Vulnerability pattern detection
├── semantic.py      # Root cause / impact / component / severity extraction
├── ir.py            # Intermediate Representation builder
├── patterns.py      # Vulnerability keyword patterns + CWE metadata
├── utils.py         # File I/O, statistics, terminal report printer
│
├── data/
│   └── cves.txt     # Input file — one CVE description per line
│
├── output.json      # Generated output (created on first run)
└── README.md        # This file
```

---

## Setup & Installation

### Requirements
- Python **3.10+** (uses `str | None` union syntax)
- No third-party packages required — pure standard library

### Clone / Download
```bash
git clone <repo-url>
cd cve_parser
```

### Prepare Input Data
Edit `data/cves.txt` and add your CVE descriptions — one per line:
```
# Lines starting with '#' are comments and will be ignored

A buffer overflow vulnerability exists in the input validation function...
SQL injection vulnerability in the login module exposes sensitive database...
```

---

## Usage

### Basic Run
```bash
python main.py
```
Reads `data/cves.txt`, writes `output.json`, and prints a full terminal report.

### Custom Input / Output
```bash
python main.py --input path/to/my_cves.txt --output results.json
```

### Quiet Mode (JSON file only, no terminal output)
```bash
python main.py --quiet
```

### Print Raw JSON to stdout
```bash
python main.py --json-only
```

### Help
```bash
python main.py --help
```

---

## Module Reference

### `lexer.py`
| Function | Description |
|---|---|
| `tokenize(text)` | Returns filtered tokens (stop-words removed) |
| `tokenize_raw(text)` | Returns all tokens without filtering |
| `clean_text(text)` | Lowercases, normalises hyphens, strips special chars |

### `parser.py`
| Function | Description |
|---|---|
| `detect_vulnerability(text)` | Returns the best-matching vulnerability type string |
| `detect_all_vulnerabilities(text)` | Returns all matching types sorted by score |

### `semantic.py`
| Function | Description |
|---|---|
| `semantic_analysis(text)` | Returns dict: `{root_cause, impact, affected_component, severity, confidence}` |

### `ir.py`
| Function | Description |
|---|---|
| `build_ir(description, vuln_type, semantics)` | Assembles enriched IR dict |
| `_extract_code_constructs(description, vuln_type)` | Heuristically finds linked code identifiers |

### `utils.py`
| Function | Description |
|---|---|
| `load_cves(filepath)` | Loads CVE descriptions from a text file |
| `save_results(results, output_path)` | Writes IR list to JSON |
| `generate_statistics(results)` | Returns aggregated stats dict |
| `print_results(results)` | Colourised terminal output of all records |
| `print_statistics(stats)` | Bar-chart style statistics summary |

### `patterns.py`
| Object | Description |
|---|---|
| `VULNERABILITY_PATTERNS` | Dict mapping vuln type → keyword phrases |
| `VULNERABILITY_METADATA` | Dict mapping vuln type → CWE ID, mitigation, references |

---

## Output Format

Each entry in `output.json` follows this schema:

```json
{
    "id": "CVE-SAMPLE-0001",
    "parsed_at": "2025-01-01T00:00:00Z",
    "description": "A buffer overflow vulnerability exists...",
    "vulnerability_type": "Buffer Overflow",
    "root_cause": "Boundary checking failure",
    "impact": "Arbitrary code execution",
    "affected_component": "Application function",
    "severity": "High",
    "confidence": 100,
    "cwe_id": "CWE-120",
    "mitigation": "Implement bounds checking, use safe string functions...",
    "references": ["https://cwe.mitre.org/data/definitions/120.html"],
    "linked_code_constructs": ["buffer", "inputValidation"]
}
```

| Field | Description |
|---|---|
| `id` | Auto-generated deterministic ID (or source CVE-ID if provided) |
| `parsed_at` | UTC timestamp of analysis |
| `vulnerability_type` | Detected class (11 categories + Unknown) |
| `root_cause` | Underlying programming flaw |
| `impact` | What an attacker achieves |
| `affected_component` | System part at risk |
| `severity` | Low / Medium / High / Critical |
| `confidence` | 0–100% estimate based on fields resolved |
| `cwe_id` | MITRE CWE identifier |
| `mitigation` | Recommended remediation |
| `references` | Links to authoritative resources |
| `linked_code_constructs` | Relevant identifiers found in description |

---

## Supported Vulnerability Types

| Vulnerability | CWE | Default Severity |
|---|---|---|
| Buffer Overflow | CWE-120 | High |
| SQL Injection | CWE-89 | Medium |
| Cross Site Scripting | CWE-79 | Medium |
| Use After Free | CWE-416 | Medium |
| Directory Traversal | CWE-22 | Medium |
| Authentication Bypass | CWE-287 | Medium |
| Integer Overflow | CWE-190 | Medium |
| Denial Of Service | CWE-400 | Medium |
| Command Injection | CWE-78 | High |
| Race Condition | CWE-362 | Medium |
| Privilege Escalation | CWE-269 | High |

---

## Sample Run

```
  Loaded 17 CVE description(s) from 'data/cves.txt' …

✔  Results written to 'output.json'

══════════════════════════════════════════════════════════════
   CVE VULNERABILITY MAPPING — PARSED RESULTS
══════════════════════════════════════════════════════════════

[01] [HIGH] Buffer Overflow
     ID          : CVE-SAMPLE-0001
     Description : A buffer overflow vulnerability exists in the input validati…
     Root Cause  : Boundary checking failure
     Impact      : Arbitrary code execution
     Component   : Application function
     CWE         : CWE-120
     Confidence  : 100%
     Constructs  : buffer
     Mitigation  : Implement bounds checking, use safe string functions, enable…

══════════════════════════════════════════════════════════════
   VULNERABILITY STATISTICS
══════════════════════════════════════════════════════════════

  Total CVEs processed: 17
  Average confidence  : 78.2%

  ── By Vulnerability Type ──────────────────────
  Denial Of Service            ███ 3
  Buffer Overflow              ███ 3
  Cross Site Scripting         ███ 3
  SQL Injection                ██ 2
  Authentication Bypass        ██ 2
  Use After Free               ██ 2
  Directory Traversal          █ 1
  Integer Overflow             █ 1

  ── By Severity ────────────────────────────────
  Medium       █████████████ 13
  High         ████ 4
```

---

## Extending the Tool

### Add a new vulnerability type
1. Add keyword phrases to `VULNERABILITY_PATTERNS` in `patterns.py`
2. Add CWE metadata to `VULNERABILITY_METADATA` in `patterns.py`
3. Optionally add semantic rules to `ROOT_CAUSE_RULES` / `IMPACT_RULES` in `semantic.py`

### Feed real CVE-IDs
Pass a `source_id` to `build_ir()` in `main.py`:
```python
record = process_cve(cve_text, source_id="CVE-2023-12345")
```

### Integrate with the NVD API
Replace `load_cves()` in `utils.py` with an HTTP fetch from
`https://services.nvd.nist.gov/rest/json/cves/2.0` to pull live CVE data.

---

## Compiler Concepts Applied

| Compiler Phase | Implementation in This Project |
|---|---|
| **Lexical Analysis** | `lexer.py` — tokenisation, stop-word removal, text normalisation |
| **Parsing / Pattern Matching** | `parser.py` — weighted keyword pattern matching, multi-type scoring |
| **Semantic Analysis** | `semantic.py` — rule-based meaning extraction (root cause, impact, severity) |
| **Intermediate Representation** | `ir.py` — structured IR dictionary as canonical internal format |
| **Source-to-Source Transformation** | CVE free text → enriched structured JSON suitable for SIEMs / ticketing systems |

---
