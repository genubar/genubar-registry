# Genubar Registry

Centralized release metadata and package catalog for the **Genubar** global menubar ecosystem:
- Core daemon, CLI, GTK4 hook, Settings UI & installer: [genubar/genubar](https://github.com/genubar/genubar)
- GNOME Shell top-panel extension (`genubar@github.com`): [genubar/genubar-gnome-extension](https://github.com/genubar/genubar-gnome-extension)

## Repository Contents

- **`app.json` (Live in `v0.1.0`):** Single source of truth checked by `genubar-packages` (`UpdateService`) and **Genubar Settings -> About & Updates** to detect core application releases and compatible `genubar-gnome-extension` updates.
- **`registry.json` (Stub in `v0.1.0`, full catalog in `v0.2.0`):** Community marketplace feed queried by **Genubar Settings -> Marketplace**. In `v0.1.0`, `registry.json` serves a valid empty catalog (`{ "entries": [] }`) so default installations return `200 OK` without errors until community packages launch in `v0.2.0`.

---

## App Release Manifest Schema (app.json)

Example structure read by `genubar_packages::models::AppReleaseManifest`:

```json
{
  "latest_app_version": "0.1.0",
  "release_date": "2026-11-16",
  "download_url": "https://github.com/genubar/genubar/releases/tag/v0.1.0",
  "notes": "Release summary shown in Genubar Settings",
  "changelog": [
    "Highlights for the core application release"
  ],
  "extension": {
    "latest_version": "0.1.0",
    "min_daemon_version": "0.1.0",
    "tested_daemon_version": "0.1.0",
    "ipc_protocol_version": 1,
    "download_url": "https://github.com/genubar/genubar-gnome-extension/releases/tag/v0.1.0",
    "raw_base_url": "https://raw.githubusercontent.com/genubar/genubar-gnome-extension/main",
    "changelog": [
      "Highlights for the GNOME Shell extension release"
    ]
  }
}
```

### Compatibility Contract

When **Genubar Settings -> About & Updates** fetches `app.json`:
1. `latest_app_version` is compared against the running `genubar-daemon` / `genubar-settings` version (`SemVer`).
2. Inside `extension`, both `ipc_protocol_version` (must match current `IPC_PROTOCOL_VERSION`) and `min_daemon_version` (must be `<= CURRENT_APP_VERSION`) are validated before allowing one-click extension downloads from `raw_base_url`.

---

## Marketplace Catalog Schema (registry.json)

`registry.json` exposes an `entries` array parsed by `RemoteRegistryCatalog`. Supported package `kind` values in `v0.1.0` clients are:
- `patch`
- `profile`
- `style`
- `icon_pack`
- `script`
- `bundle`

Each entry includes `id`, `kind`, `name`, `version`, `author`, optional `min_app_version`, `description`, `tags`, and inline TOML/CSS/Python `content` (unknown future kinds or entries with a higher `min_app_version` are displayed gracefully in older clients with an update badge).

For a complete working catalog example, see `~/.config/genubar/example_registry.json` (or [example_registry.json](https://github.com/genubar/genubar/blob/main/crates/genubar-config/defaults/example_registry.json) in the `genubar` repository).

---

## Contributing Packages (PR Guidelines)

When contributing community patches, profiles, styles, icon packs, or scripts for `v0.2.0+`:
1. Ensure every entry uses a unique lowercase `id` (`[a-z0-9_-]`), accurate `min_app_version` (e.g. `"0.1.0"`), descriptive `tags`, and valid TOML/CSS/Python inside `content`.
2. Validate JSON syntax locally before opening a PR:
   ```bash
   python3 -m json.tool app.json > /dev/null
   python3 -m json.tool registry.json > /dev/null
   ```
3. Test your entry locally in **Genubar Settings -> Marketplace** by adding a local JSON path to `registry_urls` in `~/.config/genubar/settings.toml`.

---

## Using Custom Feeds

You can point Genubar at a fork, staging branch, or local file inside `~/.config/genubar/settings.toml`:

```toml
[general]
app_update_url = "https://raw.githubusercontent.com/genubar/genubar-registry/main/app.json"
registry_urls = [
    "~/.config/genubar/example_registry.json",
    "https://raw.githubusercontent.com/genubar/genubar-registry/main/registry.json",
]
```

## License

Licensed under the [MIT License](LICENSE).
