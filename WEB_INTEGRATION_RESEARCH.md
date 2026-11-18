# Pre-Research: Connecting LawDesk Web with KattyVision Engine

---

## 1. Executive Overview

This document outlines the architectural research for connecting the **LawDesk Web Application** (`https://lawdesk-web-production.up.railway.app/filings`) with the **KattyVision Court Filing Engine** (`court_ocr`).

```
[ LawDesk Web UI ]
       │  (User uploads matter documents & selects court options)
       ▼
[ HTTP REST API / Background Worker ]
       │  (POST /api/generate-bundle)
       ▼
[ KattyVision Engine (court_ocr) ]
  └── Merge -> OCR -> Paginate -> Stamp -> Split -> Compress
       │
       ▼
[ Finished Filing Packages ] (Ready for download on LawDesk)
```

---

## 2. Interface Mapping (Frontend to Engine)

| LawDesk UI Control | KattyVision Parameter (`config.json` / CLI) | Action Performed |
|---|---|---|
| **Document Uploads** | `input_dir` | Natural sorting of files (`.pdf`, `.png`, `.jpg`, `.tiff`) |
| **Court Selection** | `case.court` (e.g., Raigad District Court, DRT) | Formats cover page & portal metadata |
| **Filing Package Split** | `split.max_pages_per_file` (e.g. 50 pages) | Slices file without splitting exhibits |
| **Portal Size Ceiling** | `compress.target_size_mb` (e.g. 19.6 MB) | Triggers adaptive compression ladder |
| **Apply Seal Checkbox** | `stamp.enabled`, `stamp.pages` | Places seal PNG as vector overlay |
| **Bates Numbering** | `pagination.enabled`, `pagination.format` | Burns official page numbers onto pages |

---

## 3. Integration Patterns

### Option A: Headless API Wrapper (FastAPI / Flask)
* Wrap `court_ocr.pipeline.Pipeline` with a lightweight Python web service deployed on Railway alongside LawDesk.
* LawDesk sends a `POST` request containing document files; the service returns downloadable bundle URLs.

### Option B: Asynchronous Queue (Redis + Celery / RQ)
* For heavy multi-hundred-page matters, LawDesk enqueues the matter ID.
* A background Python worker processes the filing packages and updates the database record when ready.
