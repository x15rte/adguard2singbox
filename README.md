# adguard2singbox

Converts the AdGuard DNS filter list into a sing-box binary rule-set (`.srs`) and publishes it to GitHub Releases.

## Use in sing-box

```json
{
  "type": "remote",
  "tag": "adblock",
  "format": "binary",
  "url": "https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs"
}
```

## Published assets

From the [`rules` release](https://github.com/x15rte/adguard2singbox/releases/tag/rules):
- [`adguarddnsfilter.srs`](https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs)
- [`adguarddnsfilter.srs.sha256`](https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs.sha256)

The workflow force-updates the `rules` tag to the current workflow commit on every run, replaces the same release assets, and marks the release as latest.

## Update schedule

GitHub Actions workflow `.github/workflows/publish-adguard-rules.yml` runs:
- Daily at `02:17` UTC (`17 2 * * *`)
- Manually via `workflow_dispatch`

## Build pipeline

Each workflow run:
1. Downloads the [AdGuard SDNS filter](https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt) with retries
2. Detects runner architecture, installs the latest matching sing-box release dynamically, and verifies the package digest before install
3. Converts `filter.txt` to `adguarddnsfilter.srs` using `sing-box rule-set convert --type adguard`
4. Verifies the output exists and is non-empty (`test -s`)
5. Generates `adguarddnsfilter.srs.sha256`
6. Force-updates tag `rules` to the current commit, then publishes both files to that release

## Verify published files

```bash
curl -L -o adguarddnsfilter.srs https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs
curl -L -o adguarddnsfilter.srs.sha256 https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs.sha256
sha256sum -c adguarddnsfilter.srs.sha256
```

