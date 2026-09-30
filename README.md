# Indic speech benchmark: shareable evidence pack

Copy the whole `SHARE_EVIDENCE_PACK_FINAL` folder to the other machine. Every `evidence_part_*.zip` is below 20 MB. Keep the ZIPs, `MASTER_MANIFEST.json`, this guide, and `SCRIPTS/` together.

## 1. Assemble and verify

Requires Python 3.11 or newer; no Python packages are needed for this step.

```bash
cd SHARE_EVIDENCE_PACK_FINAL
python3 SCRIPTS/assemble_verify.py . ASSEMBLED
```

The script checks each ZIP's SHA-256 and CRC, extracts it to `ASSEMBLED/`, then checks the size and SHA-256 of every extracted file against `MASTER_MANIFEST.json`. It refuses to overwrite an existing `ASSEMBLED/` folder.

## 2. Open the report and blind test

```bash
python3 SCRIPTS/run_blind_test.py ASSEMBLED
```

Open these addresses on the same machine:

- Report: `http://127.0.0.1:8765/BENCHMARK_REPORT.html`
- Blind listening test: `http://127.0.0.1:8765/LISTENING_REVIEW/index.html`

The server binds only to `127.0.0.1`. Stop it with Ctrl+C. The blind test contains 105 anonymous clips, one for each playable TTS model-language cell. Select a language if useful, play the clip, and give **one overall score from 1 to 5**. Scores autosave in that browser. Click **Export ratings** when finished and keep the downloaded `listening-ratings.json` file. No comments or separate quality metrics are required.

## 3. Import owner scores and rebuild

From `ASSEMBLED/`, after exporting the ratings:

```bash
cd ASSEMBLED
python3 benchmark_runtime/import_ratings.py LISTENING_REVIEW /path/to/listening-ratings.json execution/listening_ratings_current.json
python3 benchmark_runtime/final_two_table_report.py
```

Reload the report to see the imported scores. To rebuild the one-page PDF and run the report checks, install Playwright/Chromium and Poppler (`pdfinfo`), then run `make report`. The HTML rebuild and blind listening test use only Python's standard library and a browser.

## Folder map after assembly

| Path | Contents |
|---|---|
| `BENCHMARK_REPORT.html` and `.pdf` | Final one-page, two-table report. The HTML links directly to its evidence. |
| `LISTENING_REVIEW/` | Blind test page, 105 anonymous audio clips, mapping, and package manifest. Model names do not appear in the listening page. |
| `execution/fresh_current/` | Exact ASR/TTS records, waveforms, run environments, logs, and selected runtime configurations. |
| `execution/metrics_current/` | Derived metrics and streaming diagnostics. |
| `execution/sources/` and `execution/dataset_sources/` | Pinned model, licence, checkpoint, and corpus provenance. |
| `execution/runner_sources/` and `execution/retained_code/` | Executed and retained runner code. |
| `benchmark_runtime/` and `Makefile` | Report, blind test, scoring, validation, and packaging scripts. |
| `execution/FINAL_STATUS.json`, `final_report_validation.json`, `final_local_data_audit.json`, `final_shutdown_verification.json` | Final counts, report checks, local file integrity, and RunPod shutdown audit. |
| `indic_speech_benchmark_prd_v1/` | Scope and evaluation requirements. |

Trace an ASR percentage through the linked `frozen_records.jsonl`: summed `word_edits` divided by summed `reference_words` for that locale. Trace TTS timing and audio through the linked `records.jsonl` and waveform SHA-256. The report's CPU/GPU category and row-selection rules are in `benchmark_runtime/final_two_table_report.py`; publisher evidence is in `execution/sources/`. Human TTS scores are blank until the owner exports and imports ratings. Grok scores are not part of this pack or the report.

The report, its inputs, audio, and rebuild scripts are self-contained here. Model weight files are referenced by pinned publisher IDs and revisions rather than redistributed in the pack; they are needed only to rerun inference.
