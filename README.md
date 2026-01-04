
blacklist_to_mikrotik

Project Overview
- Purpose: Automatically download public IP blocklists and convert them into MikroTik RouterOS address-list scripts (.rsc). This repository uses GitHub Actions to fetch sources, transform lists into RouterOS commands, and publish release artifacts.

Features
- Automated fetching: Periodically retrieves public blocklists such as FireHOL levels and blocklist.de.
- Format conversion: Converts raw IP lists into MikroTik `address-list` commands and produces `.rsc` files ready for import.
- Release publishing: Each successful workflow run can publish a GitHub Release containing the original list and the generated `.rsc`.
- Retention control: Workflows can be configured to control how long Actions history and artifacts are retained.

Prerequisites
- MikroTik RouterOS: Able to import `.rsc` files via the console or `/import` command.
- GitHub repository permissions: Actions and Releases must be enabled for automatic publishing.

Usage
- Triggering: Workflows run on a schedule (configured in `.github/workflows/`) and can also be started manually via `workflow_dispatch`.
- Output: After a successful run, download the raw blocklist and the corresponding `.rsc` from the Release assets.
- Importing: Import the `.rsc` file into RouterOS using `import <file>.rsc` or paste its commands into the RouterOS terminal.

Workflows
- Location: `.github/workflows/`
- Examples: `FireHOL_Level2_to_Mikrotik.yml` (generates `FireHOL_Level2.rsc`), `FireHOL_Level1_to_Mikrotik.yml`, `FireHOL_Level4_to_Mikrotik.yml`, `blocklist-de_to_Mikrotik.yml`.

Customization
- Change sources or output names by editing the corresponding workflow file.
- Adjust retention or scheduling in the workflow configuration or repository Actions settings.

Contributing
- Pull requests welcome. For disruptive changes (history rewrite or format changes), open an issue first to discuss migration and coordination.

License & Support
- Follow the repository LICENSE if present; if no license is provided, assume rights are not granted.
- Use Issues for questions or feature requests.

---

See the repository root [README.md](README.md) for this file.

