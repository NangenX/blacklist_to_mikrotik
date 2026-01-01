blacklist_to_mikrotik

This repository automates downloading public IP blocklists and converting them into MikroTik RouterOS address-list scripts (.rsc). The project uses GitHub Actions to fetch sources, transform them into the MikroTik format, publish release artifacts, and maintain workflow history.

Key features
- Automatically downloads multiple public blocklists (FireHOL levels and blocklist.de).
- Converts raw IP lists into MikroTik `address-list` commands and generates `.rsc` files.
- Publishes each run as a GitHub Release containing the original `blacklist.txt` (or equivalent) and the generated `.rsc` file.
- Cleans up old GitHub Actions run history (configured to retain 7 days by default).

Workflows
- `.github/workflows/blocklist-de_to_Mikrotik.yml` — Downloads blocklist.de and generates `blocklist-de_List.rsc`.
- `.github/workflows/FireHOL_Level1_to_Mikrotik.yml` — Downloads FireHOL level1 and generates `FireHOL_Level1.rsc`.
- `.github/workflows/FireHOL_Level2_to_Mikrotik.yml` — Downloads FireHOL level2 and generates `FireHOL_Level2.rsc`.
- `.github/workflows/FireHOL_Level4_to_Mikrotik.yml` — Downloads FireHOL level4 and generates `FireHOL_Level4.rsc`.

Usage
1. The workflows run on a schedule (every 6 hours) or can be triggered manually via `workflow_dispatch`.
2. Each successful run creates/updates the corresponding `.rsc` file at the repository root and publishes a GitHub Release containing the raw list and the `.rsc` output.
3. To import the generated script into MikroTik RouterOS, download the `.rsc` from the Release and run the file in the RouterOS terminal or use the `/import` command.

Notes & recommendations
- The workflows intentionally limit stored Actions history; deleting or rewriting commits is a destructive operation and not recommended without backups.
- If you want to customize lists, modify the workflows under `.github/workflows/` and adjust the source URLs or output filenames.

Contributing
- Pull requests are welcome. For large changes (history rewrite, format changes), please open an issue first so we can discuss migration and coordination with forks and users.

License
- Licensed under the terms provided in this repository (if any). If no license is present, assume all rights reserved.

Contact
- Open an issue for questions or feature requests.

