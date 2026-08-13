# LINE standalone app (Linux)

Run the official **LINE Chrome extension** as a standalone desktop app on Linux —
its own chromeless window, taskbar/dock entry, and icon, no browser tabs or address
bar. It launches the extension's UI in `--app` mode, so the window still runs inside
the extension context (every `chrome.*` API and your LINE login keep working). It is
*not* a detached web app — LINE's UI depends on extension-only APIs
(`chrome.storage`, `chrome.cookies`, `chrome.notifications`, …) and on the extension
bypassing CORS, so running it truly outside a browser extension isn't practical.

There are two install methods. **The Chromium method is the standard, recommended
one** — it needs no Chrome Web Store install and is what's packaged as an RPM.

---

## Recommended: Chromium (self-contained, no Web Store install)

Fetches the LINE extension itself and runs it side-loaded in **Chromium**, in a
dedicated profile. No manual "Add to Chrome" step; your login persists across
restarts. (Google Chrome can't be used here — it blocks `--load-extension`; see
[`chromium/README.md`](chromium/README.md) for the full story.)

**Fedora — RPM (easiest):**

```bash
cd packaging/rpm && ./build-rpm.sh
sudo dnf install ./dist/line-standalone-chromium-*.noarch.rpm   # pulls in chromium
```

**Any distro — scripts:**

```bash
cd chromium && ./install-chromium.sh     # one-time; then launch "LINE" from your menu
```

Details, trade-offs, and uninstall: [`chromium/README.md`](chromium/README.md) ·
[`packaging/rpm/README.md`](packaging/rpm/README.md).

---

## Alternative: Google Chrome + installed extension

If you already use Google Chrome with the LINE extension installed from the Web
Store, this method reuses that install — so it **shares your Chrome profile** (same
login, working push notifications) instead of an isolated one.

Prerequisites: Google Chrome, the official
[LINE extension](https://chromewebstore.google.com/detail/line/ophjlpahpchlmihnnnihgmmeilfjmjjc)
installed and logged in.

```bash
./install.sh                        # auto-detects the Chrome profile with LINE
./install.sh --list                 # show profiles that have LINE installed
./install.sh --profile "Profile 1"  # pick a specific profile
./uninstall.sh                      # remove it
```

Overrides: `CHROME_BIN=/path/to/chrome`, `CHROME_CONFIG=~/.config/google-chrome`.

> The two methods write the **same** `.desktop` entry (same app_id), so use one at a
> time — a user-level entry shadows a system (RPM) one.

---

## Why the launcher looks the way it does (GNOME Wayland)

Under native Wayland, the browser tags an `--app=` window with a computed `app_id`:

```
chrome-<extension-id>__index.html-<Profile>
```

(e.g. `chrome-ophjlpahpchlmihnnnihgmmeilfjmjjc__index.html-Default`; a profile named
`Profile 1` becomes `Profile_1`). GNOME matches a running window to a launcher by
that `app_id`, so the `.desktop` file's **filename** and its **`StartupWMClass`**
must both equal that exact string — otherwise the window shows a generic/blank icon
instead of grouping under the launcher. The `--class` flag does *not* affect this;
it's ignored for app windows. The installers set this for you.

## Layout

- **`chromium/`** — the standard Chromium approach (fetches the extension; scripts).
- **`packaging/rpm/`** — Fedora RPM of the Chromium approach (`Requires: chromium`).
- **`install.sh` / `uninstall.sh`** — the Google Chrome installed-extension alternative.

The LINE logo is never bundled (it's LINE Corporation's trademark); it's extracted
from the fetched/installed extension at install time.
