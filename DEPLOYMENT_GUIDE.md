# AMP Customer Deployment Guide

This guide covers deploying AMP to your GitHub organization.

## Prerequisites

- GitHub organization on GitHub Enterprise Cloud (GHEC)
- GitHub PAT(s) with appropriate access to source and destination orgs
- Distribution credentials (AMP_DIST_TOKEN, AMP_VERSION, AMP_DIST_REPO)

## Quick Start

### 1. Create the Control Repository

This can be done manually in GitHub, or using the deployment script provided by your Xebia engagement contact.

```bash
# Using the deployment script (provided by Xebia)
python3 scripts/deploy-customer.py github.com/your-org/amp-control \
    --token <github-pat>
```

Or create manually:
1. Create a new private repository: `<your-org>/amp-control`
2. Copy this entire directory's contents to that repository

### 2. Install AMP CLI (Source Installs Only)

If you have source access (enterprise customers only):

```bash
pip install -e .
```

For wheel installs, the GitHub Actions workflows handle installation automatically via the `setup-amp` composite action.

### 3. Configure GitHub App (Recommended)

The platform works with personal access tokens (PATs), but GitHub Apps provide better security and audit trails.

See [docs/GITHUB_APP_SETUP.md](docs/GITHUB_APP_SETUP.md) for App creation and configuration.

### 4. Set Up Labels

```bash
amp setup-labels --repo <your-org>/amp-control \
    --token <your-github-pat>
```

This creates/updates all migration labels (status, cmd, analysis, post-migrate) in your control repo. You can then copy them to other repos or keep them centralized here.

### 5. (Optional) Create a Projects V2 Board

For visual tracking of migrations:

```bash
amp setup-project --repo <your-org>/amp-control \
    --name "Migrations" \
    --owner <your-org>
```

Then manually configure views and group-by fields in GitHub UI (API doesn't expose grouping yet).

### 6. Configure Workflows

Each workflow needs certain secrets and variables. See `.github/workflows/` for detailed requirements.

**Minimum required secrets:**
- `GH_DEST_PAT` — GitHub token for destination org (for migrations)
- `GH_PAT` — GitHub token for source (for analysis)
- `AMP_DIST_TOKEN` — Read token for wheel distribution (required)

**Minimum required variables:**
- `AMP_VERSION` — e.g., `0.1.0`
- `AMP_DIST_REPO` — e.g., `xebia/amp-dist`

### 7. Set Up Source/Target Mappings

Copy and customize the example files in `config/mappings/`:

```bash
cp config/mappings/EXAMPLE_ADO.yml config/mappings/my-org-ado.yml
# Edit my-org-ado.yml with your Azure DevOps org details
```

Reference these mappings in the `amp analyze` command:

```bash
amp analyze --source ado --org myorg --project MyProject \
    --repo <your-org>/amp-control --token <ado-pat>
```

### 8. Analyze First Batch

```bash
amp analyze --source github --org my-source-org \
    --target-org my-dest-org \
    --repo <your-org>/amp-control \
    --token <source-pat> \
    --dest-token <dest-pat>
```

This creates one issue per repository with full analysis. See [AMP_User_Guide.md](docs/AMP_User_Guide.md) for the next steps.

## Configuration

### Label Customization

Edit `config/labels.yml` and re-run `amp setup-labels` to sync changes to your repos.

### Failure Rules

Edit `config/failure_rules.yml` to customize the tier-1 failure advisor patterns.

### Projects V2 Sync

Edit `config/project_views.yml` to define custom views. Sync to GitHub:

```bash
amp configure-project --owner <your-org> --number <project-N> --force
```

## Troubleshooting

### Token Resolution

AMP tries multiple token sources in priority order. See [AGENTS.md#token-resolution](AGENTS.md) for the full fallback chain and how to debug which token is being used.

### Workflow Failures

Check the workflow logs in `.github/workflows/` runs. Common issues:
- Missing secrets/variables
- Expired or revoked tokens
- Network/API rate limits
- Source platform connectivity

### Queue Issues

If migrations get stuck in queued state, check `migration-dispatcher.yml` logs. The queue reconciler runs periodically and can be triggered manually.

## Support

For issues or questions, contact your Xebia engagement contact or open an issue in the control repository.

See [AMP_Known_Limitations.md](docs/AMP_Known_Limitations.md) for known issues and workarounds.

## Next Steps

- [AMP User Guide](docs/AMP_User_Guide.md) — Full walkthrough of the migration flow
- [Label Guide](docs/AMP_Label_Guide.md) — Detailed label taxonomy and state machine
- [GHEC-DR Runbook](docs/CLIENT_RUNBOOK_GHEC_DR.md) — If deploying to GHEC-DR
- [Known Limitations](docs/AMP_Known_Limitations.md) — Before you start
