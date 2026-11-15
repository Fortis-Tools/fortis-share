# 📤 fortis-share

**AirDrop for Linux → Android.** Send files, folders, screenshots and your clipboard from your laptop to your phone in one command, or one right-click.

```bash
share report.pdf        # → asks who's receiving → lands in your phone's Download/Share
share -c                # send whatever is on the clipboard (screenshot, copied files, text)
share -s                # send your latest screenshot
```

- **Direct to your phone** over [Tailscale](https://tailscale.com) + Termux `sshd`, so it works on any network, not just the same Wi-Fi.
- **QR fallback**: if the phone isn't reachable, or you're sending to someone else's device, it shows a QR code for a one-time download link that stops itself after the download.
- **No terminal needed**: right-click in Dolphin → **Send to phone**, or bind **Send clipboard to phone** to a key.
- **No cloud**: files go laptop → phone directly. QR codes are generated locally, and links carry a random secret token.

Part of the [Fortis](https://github.com/Fortis-OS) tools.

---

## Install

```bash
npm install -g fortis-share
share --doctor            # check dependencies
share --setup             # pair your phone (once)
share --install-desktop   # optional: Dolphin right-click + clipboard launcher
```

**Laptop (Linux):** Python 3.8+, `ssh`, `rsync`, `qrencode`. Optional: `wl-clipboard` (clipboard), `tailscale`, `libnotify-bin`, `python3-pyqt6` (GUI).

```bash
sudo apt install python3 openssh-client rsync qrencode wl-clipboard libnotify-bin python3-pyqt6
```

**Phone (Android):** [Termux](https://f-droid.org/packages/com.termux/), [Termux:API](https://f-droid.org/packages/com.termux.api/) and [Tailscale](https://tailscale.com/download/android). In Termux:

```bash
pkg install openssh rsync termux-api
termux-setup-storage                     # lets Termux write to /sdcard
# add your laptop's public key to ~/.ssh/authorized_keys, then:
sshd                                     # listens on port 8022
```

`share --setup` lists your Tailscale devices, lets you pick the phone, and tests the connection.

---

## Usage

| Command | What it does |
|---|---|
| `share <file/folder> [...]` | Ask who's receiving, then send |
| `share -c` / `--clip` | Send the clipboard: a screenshot image (Spectacle copies one), files copied in Dolphin, or text |
| `share -s` / `--shot` | Send the newest image in `~/Pictures/Screenshots` |
| `share -t phone …` | Skip the question: straight to your phone (QR if offline) |
| `share -t qr …` | Skip the question: QR for any device on the same Wi-Fi |
| `share --check` | Is the phone reachable right now? |
| `share --dest [folder]` | Show or set the folder on the phone (default `/sdcard/Download/Share`) |
| `share --setup` | Set up or change your phone |
| `share --install-desktop` | Add Dolphin's **Send to phone** and the **Send clipboard to phone** launcher |
| `share --doctor` | Check dependencies and the phone connection |

### How it decides

```
share file
  └─ Who's receiving?
       ├─ 📱 Phone ── Tailscale online? ─ sshd up? ─ key works? ── rsync → phone folder
       │                    └──────── any "no" ───────┴─→ QR fallback (tells you why)
       └─ 🌐 Other device ──────────────────────────────────→ QR
```

**Phone:** files are copied with `rsync` (resumable, with progress), then made visible in Gallery and Files (`termux-media-scan`), and you get a notification on the phone. Clipboard **text** goes straight onto the phone's clipboard.

**QR:** a single file is served as-is; folders and multiple files are zipped. The server binds a one-time URL (`/<random token>/<name>`), so anything else returns `404`. It stops 5 seconds after a complete download, or after 10 minutes. Short clipboard text is put **inside** the QR itself, so no server runs at all.

> **Firewall:** if `ufw` is on, the other device may not be able to open the QR link. `share` prints the exact allow command for your current network, e.g. `sudo ufw allow from 192.168.1.0/24 to any port 8765 proto tcp`.

---

## GUI

`share-gui` is the same engine in a small native (Qt) window: pick the receiver, see a progress bar, or get a QR window with **Copy link**. It closes itself when the transfer is done.

- **Dolphin:** right-click any file or folder → **Send to phone**
- **Launcher:** "Send clipboard to phone" (its right-click action: "Send latest screenshot to phone")
- **Shortcut:** System Settings → Keyboard → Shortcuts → *Add Application* → "Send clipboard to phone" → e.g. `Meta+Shift+P`. Then take a screenshot, press the shortcut, and it's on your phone.

---

## Config

`~/.config/fortis-share/config.json` (written by `share --setup`):

```json
{
  "phone_name": "Pixel 8",
  "phone_ip": "100.x.y.z",
  "phone_port": "8022",
  "ssh_key": "~/.ssh/id_ed25519",
  "known_hosts": "~/.ssh/known_hosts",
  "dest": "/sdcard/Download/Share",
  "qr_port": 8765,
  "qr_timeout_min": 10
}
```

**Fortis users:** set `"fortis_backup_conf": "~/path/to/fortis-ecosystem/backup/backup.conf"` and the phone's IP and port follow `fortis-backup` automatically. Set `"sshd_hint"` to change the "sshd isn't running" tip, e.g. to `on the phone: fortis-mobile start`.

---

## License

MIT © Ganesh Angadi
