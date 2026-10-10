# Perpustakaan Negara Malaysia (PNM) Harvester Runbook

> **Operational guide for harvesting and synchronizing bibliographic records from Perpustakaan Negara Malaysia.**

For universal ecosystem contracts, adapter architecture, and schema definitions, refer to the upstream specifications:
* [Master Harvester Ingestion Standards](https://github.com/pustaka-terbuka/master/blob/main/docs/HARVESTERS.md)
* [Universal Harvester Developer Guide](https://github.com/pustaka-terbuka/master/blob/main/docs/HARVESTER_DEVELOPER_GUIDE.md)
* [Chunk Lifecycle & Ephemeral Architecture](https://github.com/pustaka-terbuka/master/blob/main/docs/HARVESTER_CHUNK_LIFECYCLE.md)

---

## 1. Target System Profile

* **Institution:** Perpustakaan Negara Malaysia (National Library of Malaysia)
* **Organization Code:** `pnm` (Priority: 10)
* **Catalog Discovery:** Innovative Interfaces Vega Discover
* **Host Domain:** `opac.pnm.gov.my`
* **API Endpoint:** `https://ap.iiivega.com/api/search-result/search/format-groups`
* **Underlying Engine:** OpenSearch with `max_result_window = 10000`
* **Politeness Delay:** Minimum 2,000 ms between page requests

---

## 2. Ingestion & Conversion Rules

* **Output Inbox:** `harvest/inbox/pnm/<run-id>/`
* **MARC Format:** MARC 21 Slim XML (`<marc:collection>`)
* **Identifiers:**
  * Control number `001`: Extracted from `identifiers.local` (e.g., `vtls000437338`).
  * ISBN: Extracted from `identifiers.isbn` and `materialTabs[].identifiedBy.isbn` &rarr; `020 $a`.
  * ISSN: Extracted from `identifiers.issn` &rarr; `022 $a`.
  * Call Numbers: Extracted from `materialTabs[].editions[].callNumber` &rarr; `852 $h`.
* **Validation Gate:** Records lacking either `001` or `245 $a` are skipped and logged as errors.

---

## 3. Execution Commands

### Full Partitioned Harvest (Initial Crawl)
To harvest the entire 420k+ catalog while bypassing OpenSearch's 10,000 result ceiling:
```bash
node tools/harvesters/pnm/harvester.js --partitioned
```

### Resuming an Interrupted Run
To resume the latest incomplete run (auto-detects cursor from `partition_state.json` and scrubs any corrupted batch XML files):
```bash
node tools/harvesters/pnm/harvester.js --partitioned --resume auto
```
Or resume a specific timestamped run:
```bash
node tools/harvesters/pnm/harvester.js --partitioned --resume 2026-10-08T14-42-04Z
```

### Daily Incremental Sync (Accession Window)
Once the historical catalog is fully collected, run daily delta updates (runs in 2–5 minutes):
```bash
npm run harvest:incremental
# or:
node tools/harvesters/pnm/harvester.js --mode incremental
```

### Running Harvester Test Suite
```bash
npm test
```

---

## 4. State & Recovery

* **`partition_state.json`**: Located in the active inbox directory. Tracks completed partition keys (`year_2024_a`, etc.), `nextBatchNumber`, and `totalRecordsSaved`.
* **Self-Healing on Crash:** The engine inspects existing batch files on startup. Any file missing the closing `</collection>` tag is deleted, and its entries are scrubbed from `provenance.jsonl` before resumption begins.
