# Organization-Wide Repository Backup Process

This document outlines the high-level architecture and configuration of the automated organization-wide GitHub repository backup process for Wildlife.ai.

## Overview

We use a GitHub Actions workflow (`.github/workflows/org-backup.yml`) to automatically discover all repositories in the organization and back them up on a weekly schedule. The backup archives are compressed and uploaded securely to a Google Workspace Shared Drive.

The backup contains:
- A full Git mirror clone of the repository (`.git`).
- Metadata (issues, pull requests, releases, and repo details) exported as JSON files.

## Architecture

The backup process consists of two main jobs:

1. **Repository Discovery (`list-repos`)**
   - Authenticates using a GitHub Personal Access Token (PAT).
   - Queries the GitHub API to dynamically retrieve a list of all repositories (including private ones) belonging to the organization.
   - Outputs the list as a JSON array to be consumed by the next job.

2. **Backup Execution (`backup`)**
   - Runs a matrix strategy, executing a parallel backup task for each repository discovered.
   - Clones the repository using `--mirror` to capture all branches and tags.
   - Exports repository metadata via the `gh api`.
   - Compresses the source code and metadata into a highly efficient `.tar.zst` archive using `zstd`.
   - Uploads the archive to a Google Shared Drive using `rclone`.

## Configuration & Secrets

To function correctly, the workflow relies on two securely stored GitHub Organization/Repository Secrets. **Never commit these secrets directly into the codebase.**

### 1. `GH_BACKUP_TOKEN`
- **What it is:** A GitHub Personal Access Token (Fine-grained or Classic).
- **Purpose:** Used to authenticate `gh` CLI commands to list repositories across the organization, clone private repositories, and export metadata.
- **Permissions Required:** 
  - Organization: `Members` (Read)
  - Repositories: `Contents` (Read), `Issues` (Read), `Pull requests` (Read), `Metadata` (Read).

### 2. `RCLONE_CONFIG_CONTENTS`
- **What it is:** A JSON-embedded configuration string for `rclone`.
- **Purpose:** Authenticates `rclone` with Google Drive and specifies the destination Shared Drive.
- **Implementation Details:**
  - We use a Google Cloud **Service Account** to perform unattended uploads.
  - The Service Account must be added as a **Content Manager** to the target Google Shared Drive to prevent `storageQuotaExceeded` errors (Service Accounts have a 0-byte quota on personal drives by default).
  - The configuration uses inline JSON `service_account_credentials` to pass the Service Account key directly via an environment variable, preventing bash quoting interpolation errors during workflow execution.

### Expected Config Format (Template)

```ini
[gdrive]
type = drive
scope = drive
service_account_credentials = {"type": "service_account", "project_id": "...", "private_key_id": "...", "private_key": "...", "client_email": "...", "client_id": "...", "auth_uri": "...", "token_uri": "...", "auth_provider_x509_cert_url": "...", "client_x509_cert_url": "...", "universe_domain": "googleapis.com"}
team_drive = <SHARED_DRIVE_ID>
```
*(The entire JSON object for `service_account_credentials` must be on a single continuous line).*

## Monitoring & Recovery

- **Triggering:** The workflow runs automatically every Sunday at 00:00 UTC. It can also be triggered manually at any time via the GitHub Actions `workflow_dispatch` button.
- **Resilience:** The upload step includes a retry mechanism (up to 3 attempts with exponential backoff) to handle transient network errors. A final verification step ensures the uploaded archive exists on the remote drive before the job marks as successful.
- **Logs:** Per-repository backup status can be monitored in the GitHub Actions run logs. If a repository backup fails, the matrix strategy is configured with `fail-fast: false`, meaning other repository backups will continue uninterrupted.
