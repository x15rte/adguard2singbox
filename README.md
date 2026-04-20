# adguard2singbox

Automatically converts the AdGuard DNS filter into a sing-box binary rule-set (`.srs`) and publishes it to GitHub Releases.

## Automation

A GitHub Actions workflow at `.github/workflows/publish-adguard-rules.yml` runs:
- Daily on schedule (`17 2 * * *`, UTC)
- Manually via `workflow_dispatch`

Each run does the following:
1. Downloads [AdGuard SDNS filter](https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt)
2. Installs [sing-box](https://sing-box.app/install.sh) (beta)
3. Converts to `adguarddnsfilter.srs`
4. Generates `adguarddnsfilter.srs.sha256`
5. Publishes both files to release tag `rules`

## Published assets

From the [`rules` release](https://github.com/x15rte/adguard2singbox/releases/tag/rules):
- [`adguarddnsfilter.srs`](https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs)
- [`adguarddnsfilter.srs.sha256`](https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs.sha256)

## Reference docs
- [sing-box AdGuard rule-set documentation](https://sing-box.sagernet.org/configuration/rule-set/adguard/)
