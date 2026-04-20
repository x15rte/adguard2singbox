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

The workflow updates the same `rules` release on every run (`overwrite_files: true`) and marks it as latest (`make_latest: true`).

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
6. Publishes both files to release tag `rules`

## Run locally (optional)

```bash
curl -fL --retry 3 --retry-delay 5 -o filter.txt https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
set -euo pipefail
latest_tag=$(gh api repos/SagerNet/sing-box/releases --jq 'map(select(.draft == false)) | .[0].tag_name')
version="${latest_tag#v}"
deb_arch=$(dpkg --print-architecture)
case "$deb_arch" in
  amd64) sb_arch="amd64" ;;
  arm64) sb_arch="arm64" ;;
  *) echo "Unsupported architecture: $deb_arch"; exit 1 ;;
esac
asset="sing-box_${version}_linux_${sb_arch}.deb"
expected_sha=$(gh api repos/SagerNet/sing-box/releases/tags/"$latest_tag" --jq '.assets[] | select(.name=="'"$asset"'") | .digest' | sed 's/^sha256://')
asset_api_url=$(gh api repos/SagerNet/sing-box/releases/tags/"$latest_tag" --jq '.assets[] | select(.name=="'"$asset"'") | .url')
test -n "$expected_sha"
test "$expected_sha" != "null"
test -n "$asset_api_url"
test "$asset_api_url" != "null"
gh api -H "Accept: application/octet-stream" "$asset_api_url" > "$asset"
echo "$expected_sha  $asset" | sha256sum -c -
sudo dpkg -i "$asset"
rm -f "$asset"
sing-box rule-set convert --type adguard --output adguarddnsfilter.srs filter.txt
test -s adguarddnsfilter.srs
sha256sum adguarddnsfilter.srs > adguarddnsfilter.srs.sha256
```

## Verify published files

```bash
curl -L -o adguarddnsfilter.srs https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs
curl -L -o adguarddnsfilter.srs.sha256 https://github.com/x15rte/adguard2singbox/releases/download/rules/adguarddnsfilter.srs.sha256
sha256sum -c adguarddnsfilter.srs.sha256
```

## Troubleshooting

- If conversion fails, ensure `sing-box` is installed and available in `PATH`.
- If download fails, retry later; the source endpoint may be temporarily unavailable.
- If release upload fails, verify the workflow has `contents: write` permission.

## References

- [AdGuard SDNS filter source](https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt)
- [sing-box AdGuard rule-set documentation](https://sing-box.sagernet.org/configuration/rule-set/adguard/)
