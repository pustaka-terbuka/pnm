# Pustaka Terbuka — PNM Data Provider

> **PustakaTerbuka: an open, shared MARC catalog for Malaysian schools and libraries.**  
> *Official Perpustakaan Negara Malaysia (PNM) Bibliographic Dataset and Harvester for [Pustaka Terbuka](https://github.com/pustaka-terbuka).*

This repository contains the complete open MARCXML bibliographic catalog harvested from **Perpustakaan Negara Malaysia (PNM)**, sharded for Git distribution and consumption by the Pustaka Terbuka Union Catalog ([`pustaka-terbuka/master`](https://github.com/pustaka-terbuka/master)).

---

## Architecture

The PNM repository functions as the upstream national library authority data provider within the Pustaka Terbuka federated catalog architecture. Ingested records flow to the Master Hub:
- **Central OPAC & Search Portal:** [github.com/pustaka-terbuka/master](https://github.com/pustaka-terbuka/master)
- To view an organization's collection on the master portal: `https://pustaka-terbuka.github.io/master/?org=PNM`

## Quick Start for Librarians

Librarians can use this dataset for copy cataloging and finding PNM records:
- Individual MARCXML records can be retrieved at `data/bib/{shard}/{recordId}.xml`.
- Physical call numbers and holdings are tracked in `data/holdings.csv`.


---

## Catalog Overview

- **Total Records:** 422,186 MARCXML records
- **Organization Code:** `pnm`
- **Source Priority:** 10 (National Library primary authority)
- **Directory Structure:** Sharded under `data/bib/{shard}/{recordId}.xml` (`00/` through `zz/`)
- **Checksum Manifest:** `data/manifest.json` tracks SHA-256 hashes of every bibliographic XML file
- **Lookup Aliases:** `data/aliases.jsonl.gz` indexes ISBN, ISSN, and original control numbers (`001`) to canonical IDs
- **Physical Holdings:** `holdings.csv` contains PNM item barcodes, call numbers, and collection locations

---

## Repository Structure

```text
pnm/
├── data/
│   ├── bib/                # 422,186 MARCXML records sharded 00/ to zz/
│   ├── holdings.csv        # PNM physical item holdings & shelf call numbers
│   ├── manifest.json       # SHA-256 integrity manifest for all records
│   ├── aliases.jsonl.gz    # ISBN/ISSN/001 lookup table
│   └── sources.json        # Provenance attribution definition
├── harvest/                # Automated PNM OpenSearch harvester scripts & logs
├── tools/                  # Harvester utilities and validation scripts
└── docs/                   # Harvester chunk lifecycle & ingestion standards
```

---

## Harvesting & Updates

The PNM collection was harvested via the official PNM OpenSearch catalog interface using chunk-based harvesting with rate limiting and exponential backoff.

Harvester configuration and scripts:
- `harvest/` — Harvesting scripts and configuration
- `tools/harvesters/pnm/` — OpenSearch crawler and MARCXML converter
- `docs/HARVESTER.md` — Institution runbook and execution commands
- `docs/HARVESTERS.md` — Ingestion standards and tag preservation contracts
- `docs/HARVESTER_CHUNK_LIFECYCLE.md` — Batch harvest lifecycle documentation

To validate harvester logic or run an incremental harvest:
```powershell
npm test                  # Validate harvester logic
node tools/harvesters/pnm/harvester.js --limit 1000 --out harvest/inbox/
```

---

## Integration with Master Hub

This repository functions as the primary national data provider for the **Pustaka Terbuka Master Hub**:
- **Central OPAC & Search Portal:** [github.com/pustaka-terbuka/master](https://github.com/pustaka-terbuka/master)
- The Master Hub ingests records from this repository, performs cross-institutional deduplication, generates Pagefind static search indices, and deploys the unified OPAC.

---

## License & Attribution

- **Bibliographic Metadata:** Public Domain / Open Data under [ODC-By 1.0](docs/DATA_LICENSE.md).
- **Original Source Authority:** Perpustakaan Negara Malaysia (National Library of Malaysia).
- Records preserve original control numbers (`001`, `035`) and cataloging source tags (`040 $a MY-KLP`).
