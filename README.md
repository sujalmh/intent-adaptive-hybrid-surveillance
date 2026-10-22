# Intent-Adaptive Hybrid Retrieval and Heterogeneous-Space Person Fusion for Conversational Video Surveillance (INDICON 2026)

Public research artifact: reproducible code, tests, eval harness, benchmarks and measured results for a conversational video-surveillance system. Three contributions: **(C1)** a verified alert compiler (deterministic 9-tuple rule materializer + clarify/abstain validator), **(C2)** a heterogeneous-space person descriptor (weighted concatenation of frozen SigLIP + MiniLM encoders), **(C3)** an intent-adaptive retrieval routing rule with hard overrides.

## Layout

- `backend/app/` — library code imported by the tests and harness (retrieval, fusion, alert grammar/validator, eval helpers). Unimported API/router/production modules are deliberately excluded.
- `backend/tests/` — full test suite (standalone green run, no MongoDB, no network).
- `backend/scripts/` — the eval scripts (`eval_*.py`: ablations, alert/fusion/retrieval controls, calibration and probe variants with pre-registered frozen winners) plus the latency harness (`measure_latency.py`) and the test-dataset builder (`create_test_dataset.py`).
- `backend/benchmarks/` — 150 retrieval queries + protocol-matched control, tune/test splits, 2,016 person pairs, 120 alert utterances, schemas and dataset build scripts. No parse caches (regenerable).
- `backend/requirements.txt` — pinned dependencies for tests + harness.
- `data/index/` — shipped FAISS indices + metadata only (scene `faiss/`, person `faiss_person/`).
- `results/` — every measured output: `results.json` files, `tables.md` summaries, per-query CSVs and the release manifest.

Deliberately absent: frontend, paper sources/drafts, model weights, recordings/snapshots, secrets, parse caches.

## Reproduce

```powershell
pip install -r backend/requirements.txt
$env:PYTHONIOENCODING='utf-8'
python -m pytest backend/tests -q
python backend/scripts/eval_alerts.py
python backend/scripts/eval_fusion.py --seeds 5 --n-boot 3000
python backend/scripts/eval_retrieval_control.py --n-boot 3000
```

Model weights for the text encoders resolve from the local Hugging Face cache (offline); no download and no MongoDB are required. Headline numbers land in `results/` (`retrieval_control_results.json`, `fusion_results.json`, `alert_results.json`).

## Scope note (single camera)

Everything reproducible here runs on the released **single-camera** subset (camera_0, ~2 h window, 121 indexed clips / 242 scene frames / 1,778 person descriptors). Same-track labels are deterministic pseudo-labels from detector/attribute metadata (no visual cosine in formation), not human identities; cross-camera retrieval and identity tracking are untestable from this artifact. The paper's full-deployment figures (4 cameras, 312k descriptors) are recorded measurements, not reproducible outputs.
