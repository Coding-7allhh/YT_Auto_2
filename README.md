# YT_Auto_2 — Part 2 GitHub Actions runner

This repository runs `Part_2.ipynb` on a GitHub-hosted Ubuntu runner. It downloads two shared Google Drive folders, creates 80-second MP4 clips from videos in the `download` folder, and publishes the clips plus a JSON report as a GitHub Actions artifact.

## Required repository secrets

Open **Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret name | Value |
| --- | --- |
| `DRIVE_ASSETS_URL` | Share link for Google Drive folder **assets** (folder 1) |
| `DRIVE_DOWNLOAD_URL` | Share link for Google Drive folder **download** (folder 2; contains the source videos) |

For both Drive folders, set General access to **Anyone with the link — Viewer**. Paste the complete folder URL as the secret value. A bare folder ID is also accepted.

Do not paste either Drive URL in the notebook, workflow YAML, README, or source code. Do not commit secrets or downloaded media.

## Run

1. Add the two secrets above.
2. Open **Actions → Part 2 - Drive Download and Video Clips**.
3. Select **Run workflow**.
4. When complete, open the run and download the `part2-video-clips-*` artifact.

The workflow also runs when `Part_2.ipynb`, `requirements.txt`, or this workflow changes on the `main` branch.

## GitHub Actions compatibility notes

- FFmpeg is installed by the workflow using `sudo apt-get`; the notebook no longer runs `apt-get` or installs packages itself.
- Paths are workspace-relative instead of hard-coded `/content/...` Colab paths.
- The `assets` folder downloads to `assets/`; the `download` folder downloads to `download/` and is scanned recursively for videos.
- Generated clips are uploaded as an Actions artifact, not committed to Git. This avoids bloating the repository and running into Git's large-file limits.
- Standard GitHub-hosted runners are CPU-only. Parallel video workers are set to 2 by default; change `MAX_PARALLEL_VIDEOS` in the workflow only if runner memory/CPU and disk allow it.
- Large source folders (especially multi-GB videos) can exceed the disk space or runtime available on a standard hosted runner. In that case, use a self-hosted runner with sufficient free disk space and set `runs-on: self-hosted`.
