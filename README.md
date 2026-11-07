# YT Auto Master Pipeline — Local Workspace Only

This workflow runs Part 2 → Part 3 → Part 4 on a GitHub-hosted `ubuntu-latest` runner. It does **not** download from Google Drive, connect to Google Sheets, upload to Drive, or upload a GitHub Actions artifact.

## Repository layout

```text
repository/
├── download/                 # Put source videos here, e.g. download/category/video.mp4
├── assets/                   # Put logos, overlays, images, etc. here
├── Part_2.ipynb
├── Part_3.ipynb
├── Part_4.ipynb
├── requirements.txt
├── part3-requirements.txt
├── part4-requirements.txt
└── .github/workflows/yt-auto-master.yml
```

## GitHub Actions secrets

No Google secrets are needed. Add at least one AI provider key if Part 4 is expected to generate metadata:

- `OPENROUTER_API_KEY` or
- `GROQ_API_KEY`

Optional: `GEMINI_API_KEY` for Part 3's AI overlay text. Part 3 has fallback text when it is absent. These are API credentials, not cloud-storage credentials.

## How to run

1. Put source videos in `download/<CATEGORY>/`.
2. Put Part 3 assets in `assets/`.
3. Commit and push the folders and notebooks to your repository.
4. Open **Actions → YT Auto Master Pipeline — Local Workspace Only → Run workflow**.
5. Choose worker count and run.

## Outputs created during the job

- `clips/` — Part 2 clips
- `Ready_!/` — Part 3 edited videos
- `Part_4_Output/metadata_rows.csv` — local metadata report
- `Part_4_Output/run_summary.json` — local run summary
- `Part_4_Output/thum/` — local thumbnails
- `executed/` — executed notebook copies
- `logs/` — stage logs
- `run_manifest.json` — local file inventory

## Critical: GitHub-hosted runner storage is temporary

All outputs above exist only in the runner workspace while the job is running. GitHub destroys the hosted runner and its workspace after the job ends. Because this workflow intentionally has no artifact upload and no cloud-storage destination, generated outputs are **not retained or available for download after the job**. The run summary and logs shown in the Actions interface do not preserve the actual video files.

A GitHub-hosted runner also cannot see files on your phone or PC. Inputs must be present in the checked-out repository. GitHub blocks individual Git files larger than 100 MB; very large video files (such as 5–20 GB) cannot practically be committed to a normal Git repository. With the user's no-cloud-storage requirement, this setup is therefore suitable only for small inputs that can be stored in the repository and outputs that are needed only during the job.

If you later need to keep output videos after a run, you must choose a persistence method (for example, download an Actions artifact or use a self-hosted runner with a persistent disk). This package deliberately does neither.
