# biocli Pro — Biomedical Research FastMCP Server + 6 Production Agent Skills (v1.0.0)

> Turn **Claude Desktop, Cursor, Claude Code, Codex, or Antigravity** into an audited, reproducible biomedical research workstation — with **zero hallucinated PMIDs**, **24-hour local SQLite caching**, and **one-click publication-ready `methods.md` + `manifest.json` generation**.

Built by **Yangfei (`youngfly93`)**, creator of the open-source [`@yangfei_93sky/biocli`](https://github.com/youngfly93/biocli) CLI and [`pdfanno`](https://github.com/youngfly93/pdfanno).

---

## Why Researchers & Bioinformaticians Use `biocli Pro`

| Problem in Raw LLM Chats | How `biocli Pro` Solves It |
| :--- | :--- |
| LLMs hallucinate gene coordinates, UniProt IDs, and fake PMIDs | Live structured queries across **NCBI Gene, PubMed, UniProt, Ensembl, Open Targets, KEGG, STRING, Enrichr, cBioPortal & GDSC** |
| Copy-pasting across 8 browser tabs breaks focus and loses parameters | Single MCP workflow returns Markdown dossiers or JSON envelopes directly inside Claude / Cursor |
| Reviewers ask *"Which database versions and parameters did you use?"* | Every tool call is automatically logged; `generate_methods_appendix` outputs journal-ready **`methods.md`** + SHA-256 **`manifest.json`** |
| Repeat queries hit NCBI / STRING rate limits | Built-in **24h local SQLite cache** + polite per-host rate limiter (`NCBI_API_KEY` aware) |
| Generic AI coding prompts produce ugly R plots & leaky survival models | Includes **6 battle-tested `SKILL.md` SOPs** (TCGA LASSO-Cox, AutoDock Vina + PyMOL, Nature/Cell figures, scRNA-seq QC, Methods Auditor, Client Packager) |

---

## Architecture Overview

```text
Claude Desktop / Cursor / Codex / Antigravity
        │  (MCP stdio or Streamable HTTP)
        ▼
┌──────────────────────────────────────────────────────────┐
│                 biocli-pro-mcp (FastMCP)                 │
│  Tools:                                                  │
│   1. gene_dossier               4. drug_sensitivity_lookup│
│   2. disease_target_brief       5. literature_digest_template│
│   3. enrichment_pathway_check   6. generate_methods_appendix │
│  Resources: biocli://session/manifest, biocli://cache/stats  │
│  Prompt:    target_validation_dossier                    │
├──────────────────────────────────────────────────────────┤
│  24h SQLite Cache + Per-Host Rate Limiter + Ledger       │
├──────────────────────────────────────────────────────────┤
│  Dual Backend:                                           │
│   • Optional @yangfei_93sky/biocli CLI bridge (auto)     │
│   • Native async REST/GraphQL clients (zero Node needed) │
└──────────────────────────────────────────────────────────┘
```

---

## Quickstart (Under 60 Seconds)

### Option A: One-Command Installer (Recommended)

**macOS / Linux:**
```bash
chmod +x install.sh
./install.sh
```

**Windows (PowerShell):**
```powershell
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

This installs the `biocli-pro-mcp` command, verifies your setup with `--doctor`, copies the **6 Biomedical Agent Skills** into `~/.agents/skills/` (or `~/.claude/skills/`), and prints the exact JSON snippet for Claude Desktop / Cursor.

### Option B: Manual Install (`uv` or `pip`)

```bash
# With uv (recommended)
uv tool install .

# Or with standard Python 3.10+ venv
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
```

Verify installation:
```bash
biocli-pro-mcp --version
biocli-pro-mcp --doctor
```

---

## Connect to Your AI Client

### Claude Desktop
Copy `claude_desktop_config.example.json` into:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "biocli-pro": {
      "command": "biocli-pro-mcp",
      "args": ["--transport", "stdio", "--backend", "auto"],
      "env": {
        "NCBI_EMAIL": "your.name@institution.edu",
        "NCBI_API_KEY": ""
      }
    }
  }
}
```

### Cursor / Windsurf
Copy `cursor_mcp.example.json` into `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (global) and update the path or binary name.

### Streamable HTTP (Team / Local Web Server Mode)
```bash
biocli-pro-mcp --transport streamable-http --host 127.0.0.1 --port 8765
```

---

## MCP Tools Reference

| Tool Name | Upstream Databases | Output Highlights |
| :--- | :--- | :--- |
| **`gene_dossier`** | NCBI Gene, UniProtKB, Ensembl, PubMed | Cross-DB IDs, genomic coordinates, reviewed protein function, subcellular localization, recent papers |
| **`disease_target_brief`** | Open Targets GraphQL, KEGG, STRING | Modality tractability (SM/Ab/PROTAC), ranked disease associations, clinical drugs, KEGG links, PPI partners |
| **`enrichment_pathway_check`** | Enrichr, STRING | Multi-library FDR-filtered pathways (KEGG/Reactome/GO/MSigDB), odds ratio, combined score, cross-validation |
| **`drug_sensitivity_lookup`** | GDSC2 benchmarks, Open Targets, cBioPortal | Actionable oncology drug sensitivity/resistance markers, clinical pipeline drugs, cohort mutation prevalence |
| **`literature_digest_template`** | NCBI PubMed E-utilities (or offline records) | Citation integrity audit (0 fabricated PMIDs), evidence-level grading matrix, extraction scaffold, Vancouver/APA/BibTeX |
| **`generate_methods_appendix`** | Active Session Provenance Ledger | Journal-ready `methods.md` (Nature/Cell/PLOS/Client Report) + cryptographic SHA-256 `manifest.json` |

All data tools accept `response_format="markdown"` (default) or `response_format="json"`, plus `force_refresh=True` to bypass the 24-hour SQLite cache.

### Built-in MCP Resources & Prompt
- `biocli://session/manifest` — Live JSON audit trail of the current research session.
- `biocli://cache/stats` — Local SQLite cache hit/miss counters and disk usage.
- Prompt `target_validation_dossier(gene, disease)` — One-click 6-step oncology target validation SOP.

---

## Included: 6 Production Biomedical AI Agent Skills (`skills/`)

Drop any folder under `skills/` into `~/.agents/skills/`, `~/.claude/skills/`, or `.cursor/rules/`:

1. **`tcga-prognostic-blueprint`** — Leakage-free Bulk RNA-seq / TCGA / GEO LASSO-Cox prognostic signature, time-dependent ROC, calibration & Nomogram R blueprint.
2. **`molecular-docking-sop`** — End-to-end AutoDock Vina + `fpocket` binding-cavity prediction + PyMOL 300 DPI publication-style 3D/2D rendering SOP.
3. **`sci-figure-polish`** — Nature/Cell/Science aesthetic upgrade & `ggplot2` / `ComplexHeatmap` refactoring checklist (colorblind-safe palettes, typography, vector export).
4. **`single-cell-qc-annotator`** — Scanpy / Seurat v5 single-cell QC, doublet removal (`Scrublet`/`DoubletFinder`), batch integration, and evidence-backed marker annotation SOP.
5. **`bio-methods-auditor`** — Manuscript & code reproducibility gatekeeper: statistical test verification, multiple-testing audit, seed/version checks, and fake-citation scanner.
6. **`bio-client-deliverable-packager`** — Sanitized 5-folder client-ready bioinformatics delivery & MD5 checksummed ZIP packaging workflow.

---

## Environment Variables (All Optional)

| Variable | Purpose | Default |
| :--- | :--- | :--- |
| `NCBI_API_KEY` | Increases NCBI E-utilities rate limit from 3 req/s to 10 req/s | unset |
| `NCBI_EMAIL` | Polite researcher contact header recommended by NCBI | unset |
| `BIOCLI_PRO_CACHE_DIR` | Custom directory for 24h SQLite cache (`cache.sqlite3`) | `~/.cache/biocli-pro-mcp` |

---

## Running the Test Suite

### Offline Deterministic Tests (Default — 100% Mocked, Zero Network Required)
```bash
pytest -v
```

### Opt-In Live Smoke Tests (Queries Real Public APIs)
```bash
pytest -v --live -m live
```

---

## Research Use & Disclaimer

`biocli-pro-mcp` and the accompanying Agent Skills are designed strictly for academic, preclinical, and computational research productivity. Outputs are **not** intended for clinical diagnosis or patient treatment decisions. Always independently verify biological interpretations, statistical assumptions, and primary references prior to publication.
