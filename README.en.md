# blacklist_to_mikrotik

[简体中文](README.md) | **English**

Turns public IP blocklists and regional IP ranges into MikroTik RouterOS `address-list` scripts (`.rsc`). GitHub Actions fetches the sources on a schedule, converts them, and publishes the result as release assets, so a router can update itself with `/tool fetch` + `/import`.

<p>
  <a href="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/FireHOL_Level1_to_Mikrotik.yml"><img src="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/FireHOL_Level1_to_Mikrotik.yml/badge.svg" alt="FireHOL Level1"></a>
  <a href="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/FireHOL_Level2_to_Mikrotik.yml"><img src="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/FireHOL_Level2_to_Mikrotik.yml/badge.svg" alt="FireHOL Level2"></a>
  <a href="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/FireHOL_Level4_to_Mikrotik.yml"><img src="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/FireHOL_Level4_to_Mikrotik.yml/badge.svg" alt="FireHOL Level4"></a>
  <a href="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/blocklist-de_to_Mikrotik.yml"><img src="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/blocklist-de_to_Mikrotik.yml/badge.svg" alt="blocklist.de"></a>
  <a href="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/China_IP_to_Mikrotik.yml"><img src="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/China_IP_to_Mikrotik.yml/badge.svg" alt="China IP"></a>
  <a href="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/Sichuan_IP_to_Mikrotik.yml"><img src="https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/Sichuan_IP_to_Mikrotik.yml/badge.svg" alt="Sichuan IP"></a>
  <a href="https://github.com/NangenX/blacklist_to_mikrotik/releases"><img src="https://img.shields.io/github/release-date/NangenX/blacklist_to_mikrotik?label=last%20build" alt="last build"></a>
</p>

## Available lists

Each list maps to a release under a fixed tag. Asset filenames never change, so the download URLs stay stable forever.

| List | Source | Refresh | Asset | Address-list name |
| --- | --- | --- | --- | --- |
| FireHOL Level 1 | [iplists.firehol.org](https://iplists.firehol.org/) | every 6 hours | `FireHOL_Level1.rsc` | `blocklist` |
| FireHOL Level 2 | [iplists.firehol.org](https://iplists.firehol.org/) | every 6 hours | `FireHOL_Level2.rsc` | `blocklist` |
| FireHOL Level 4 | [iplists.firehol.org](https://iplists.firehol.org/) | every 6 hours | `FireHOL_Level4.rsc` | `blocklist` |
| blocklist.de | [lists.blocklist.de](https://lists.blocklist.de/lists/all.txt) | every 6 hours | `blocklist-de_List.rsc` | `blocklist` |
| China IP ranges | [china-operator-ip](https://github.com/gaoyifan/china-operator-ip) | monthly, on the 1st | `CN-IP.rsc` | `CN-IP` |
| Sichuan IP ranges | [iplist](https://github.com/metowolf/iplist) | monthly, on the 1st | `Sichuan.rsc` | `Sichuan` |

> The four blocklists all write into the same address list named `blocklist`, so only one of them can be imported at a time. To run several in parallel, copy a workflow and change the `list=` value.

### Download URLs

```
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level1-Latest/FireHOL_Level1.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level2-Latest/FireHOL_Level2.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level4-Latest/FireHOL_Level4.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/blocklist-de-latest/blocklist-de_List.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/CN-IP-Latest/CN-IP.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/Sichuan-IP-Latest/Sichuan.rsc
```

## Quick start

### 1. Import once by hand

```rsc
/tool fetch url="https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level1-Latest/FireHOL_Level1.rsc" dst-path="FireHOL_Level1.rsc" mode=https
/ip firewall address-list remove [find list=blocklist]
/import file-name=FireHOL_Level1.rsc
```

You can also drag the `.rsc` file into WinBox's Files panel and run `/import file-name=FireHOL_Level1.rsc` in the terminal.

### 2. Keep it updated (RouterOS 7.x)

```rsc
/system script add name=fetch-firehol-l1 policy=read,write,policy,test source={
    /tool fetch url="https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level1-Latest/FireHOL_Level1.rsc" dst-path="FireHOL_Level1.rsc" mode=https
    /ip firewall address-list
    :if ([:len [find list=blocklist]] > 0) do={ remove [find list=blocklist] }
    /import file-name=FireHOL_Level1.rsc
}

/system scheduler add name=firehol-l1 interval=6h on-event=fetch-firehol-l1 policy=read,write,policy,test
```

Flushing the list before importing avoids stale entries that upstream has already removed. Logging the entry count with `:log info` before the import makes troubleshooting much easier.

## Data freshness

- The release publish date is refreshed on every build, so the date shown on the Releases page is the last successful build.
- Each release description states the build time (UTC) and the number of entries, e.g. `自动构建（UTC）：2026-11-03 03:23 / 条目数：4648` (the body text itself is in Chinese).
- The first line of every `.rsc` is a comment carrying the same timestamp:

  ```
  # FireHOL Level1 | updated: 2026-11-03 03:23 UTC
  /ip firewall address-list
  add list=blocklist address=0.0.0.0/8 comment="Blocklist1"
  ```

  After importing, check the first line on the router with `/file print detail` or in any text editor to confirm the data is current.

## How it works

Every workflow runs the same four steps:

1. Fetch the raw list with `curl`, dropping comment and blank lines.
2. Convert each IP/CIDR into `add list=... address=... comment="..."` with `awk`, writing a timestamped header into the `.rsc`.
3. Delete the existing release on the fixed tag and recreate it with the new asset, so the release date moves forward while **filenames and download URLs stay identical**.
4. Purge Actions run history older than 7 days.

| Workflow | Trigger | Purpose |
| --- | --- | --- |
| `FireHOL_Level1/2/4_to_Mikrotik.yml` | every 6 hours | the three FireHOL levels |
| `blocklist-de_to_Mikrotik.yml` | every 6 hours | full blocklist.de feed |
| `China_IP_to_Mikrotik.yml` | monthly, on the 1st | Chinese ISP IP ranges |
| `Sichuan_IP_to_Mikrotik.yml` | monthly, on the 1st | Sichuan province IP ranges |
| `clean.yml` | manual | squash repository history into a single commit |
| `clean_release.yml` | manual | delete releases and tags according to `releases_keep_latest` — **it will remove the other lists' releases, run with care** |

Any workflow can also be started manually from the Actions tab.

The status badges at the top show the latest run of each workflow on `main`, so a broken feed is visible at a glance.

## Customizing

- **Change the schedule**: edit the `cron` entry (UTC). `0 */6 * * *` is every 6 hours, `0 0 1 * *` is 00:00 UTC on the 1st of each month.
- **Rename the address list**: change `list=blocklist` in the `awk` command and the output filename together.
- **Change the source**: replace the `curl` URL. Any plain-text list with one CIDR per line works.
- **Fork does not publish**: go to Settings → Actions → General → Workflow permissions and pick *Read and write permissions*, otherwise `GITHUB_TOKEN` cannot create releases.

## FAQ

**The address list is empty after import.**
Check whether `/tool fetch` actually succeeded (`/file print` and look at the file size). RouterOS versions before 7 handle the 302 redirect poorly; upgrade to 7.x.

**Fewer entries than the upstream file.**
Comment lines and blanks are stripped during conversion, so only usable IP/CIDR entries remain. That is expected.

**Is FireHOL Level 4 too heavy?**
Level 4 is close to 180k entries, which costs CPU and memory both on import and on matching. On home hardware, start with Level 1.

**Why does the release date keep changing?**
By design: each build recreates the release to refresh its publish date while filenames and URLs stay stable. See "How it works" above.

## Disclaimer

- All data comes from the third-party sources listed above; this repository only reformats it and makes no guarantee about accuracy or timeliness.
- Blocklists can block legitimate traffic. Validate in a lab before rolling out to production.
- This repository declares no open-source license yet; open an issue before reusing the code. Copyright of the upstream data belongs to its respective sources.

## Contributing

Issues and pull requests are welcome. For disruptive changes (history rewrites, publishing format changes), please open an issue first.
