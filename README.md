# YT Auto Master Pipeline — Part 2 → Part 3 → Part 4

This package consolidates the three existing notebook stages into **one manually started GitHub Actions workflow**. It runs all stages in sequence on the same runner and uploads one final artifact.

## 1. Repository layout

Copy these files into the root of your repository:

```text
repository/
├── Part_2.ipynb
├── Part_3.ipynb
├── Part_4.ipynb
├── requirements.txt
├── part3-requirements.txt
├── part4-requirements.txt
└── .github/
    └── workflows/
        └── yt-auto-master.yml
```

Do not put the notebooks inside a subfolder. The workflow expects the exact names and paths shown above.

## 2. GitHub Secrets

Open **Repository → Settings → Secrets and variables → Actions** and add:

| Secret | Required | Used by |
|---|---:|---|
| `DRIVE_ASSETS_URL` | Yes | Part 2 downloads assets; Part 3 reuses them |
| `DRIVE_DOWNLOAD_URL` | Yes | Part 2 source-video folder |
| `GEMINI_API_KEY` | No | Part 3 AI overlay text; fallback text is used if absent |
| `OPENROUTER_API_KEY` | One AI key required | Part 4 metadata generation |
| `GROQ_API_KEY` | One AI key required | Part 4 fallback metadata generation |
| `GOOGLE_SERVICE_ACCOUNT_JSON` | Yes | Part 4 Google authentication |
| `GOOGLE_DRIVE_REPORTS_FOLDER_ID` | Yes | Part 4 Google Sheets/report folder |
| `GOOGLE_DRIVE_THUMBNAIL_FOLDER_ID` | No | Optional separate thumbnail folder |

At least one of `OPENROUTER_API_KEY` or `GROQ_API_KEY` is required.

Both Google Drive source folders must be shared so the download method can access them. For Part 4, enable Google Drive API and Google Sheets API in Google Cloud, share the report/thumbnail folders with the service account email, and give it Editor access. Never commit API keys or the service-account JSON into the repository.

## 3. Run it

1. Open the repository on GitHub.
2. Select **Actions → YT Auto Master Pipeline (Part 2 → Part 3 → Part 4)**.
3. Select **Run workflow**.
4. Choose the runner, worker count, source-data archival option, and retention period.
5. Start the run.

The stages execute in this order:

1. **Part 2** downloads Drive input/assets and writes `clips/`.
2. **Part 3** reuses Part 2's assets, edits `clips/`, and writes `Ready_!/`.
3. **Part 4** reads `Ready_!/`, creates metadata, updates Google Sheets, and uploads thumbnails to Drive where configured.
4. The final upload saves all local source/intermediate/final data, executed notebooks, logs, and a `run_manifest.json`.

The full artifact is named `yt-auto-full-run-<run_number>-attempt-<attempt>`. If you turn off source-data archival, the artifact is named `yt-auto-processed-run-...` and excludes the original `download/` and `assets/` folders.

## 4. Important runner and storage limits

This is a video-heavy pipeline. The standard `ubuntu-latest` GitHub-hosted runner has **14 GB SSD storage**. It is not suitable for a source video/batch larger than the available disk once downloads, generated clips, edited videos, and temporary files are all present. For large batches, select `self-hosted` only after registering a Linux x64 self-hosted Actions runner with enough free disk space. Install FFmpeg, ffprobe, and `fonts-dejavu-core` on that machine first.

GitHub Free includes **500 MB of Actions artifact storage**, shared with GitHub Packages. A full archive containing multi-GB videos will exceed that included allowance; artifact upload may require paid storage or a different storage target. Artifact retention controls how long artifacts are kept; it does not increase your storage quota. The workflow defaults to 7 days and sets video compression to level 0 to avoid wasting CPU trying to compress already-compressed videos.

For large 5–20 GB source videos, the practical choice is a sufficiently large self-hosted runner plus a cloud/object-storage destination for large media. GitHub Actions artifacts are convenient for reports and smaller outputs, but are not a durable multi-terabyte video-storage system.

## 5. Remove the old per-stage workflows

To avoid confusion, disable or delete the separate workflow files from earlier packages:

- `.github/workflows/part2-github-actions.yml`
- `.github/workflows/part3-github-actions.yml`
- `.github/workflows/part4-github-actions.yml`

Use only `yt-auto-master.yml` to start the complete pipeline. The old notebooks and their requirements are retained in this integrated package.

## 6. What is preserved in the final artifact?

When `archive_source_data` is enabled, the artifact includes:

- `download/` — original files downloaded by Part 2
- `assets/` — shared image/video/logo assets
- `clips/` — Part 2 generated clips
- `Ready_!/` — Part 3 edited videos
- `Part_4_Output/` — metadata CSV and JSON summary
- `executed/` — executed copies of all three notebooks
- `logs/` — separate logs for all stages
- `run_manifest.json` — file inventory, byte sizes, run ID, and stage outcomes

The workflow always attempts to upload diagnostics and all files created up to the point of a failure. A failed stage therefore stops later stages but still triggers the artifact upload step.
