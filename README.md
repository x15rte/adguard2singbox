# adguard2singbox

Automatically converts the AdGuard DNS filter into a sing-box binary rule-set (`.srs`) and publishes it to GitHub Releases.

## Automation

A GitHub Actions workflow at `.github/workflows/publish-adguard-rules.yml` runs:
- Daily on schedule (`17 2 * * *`, UTC)
- Manually via `workflow_dispatch`

Each run does the following:
1. Downloads `filter.txt` from AdGuard
2. Installs sing-box (beta)
3. Converts to `adguarddnsfilter-latest.srs`
4. Generates `adguarddnsfilter-latest.srs.sha256`
5. Publishes both files to release tag `rules-latest`

## Published assets

From the `rules-latest` release:
- `adguarddnsfilter-latest.srs`
- `adguarddnsfilter-latest.srs.sha256`
