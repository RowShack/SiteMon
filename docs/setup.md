# SiteMon setup guide

This takes you from nothing to a wallboard showing your MonStack apps. Back to the [README](../README.md).

SiteMon does not talk to network gear and has no vendor account to set up. It consumes **cards**: a small JSON summary that
each MonStack app (FortiMon, UniMon, MerakiMon) serves. A key for a card can read the card and nothing else. There are two
ways to get a card to SiteMon, and you can mix them:

- **Pull.** SiteMon reads the app. Use this when SiteMon can reach the app's address.
- **Push.** The app sends its card to SiteMon. Use this when SiteMon cannot reach the app (another network, behind NAT).

## Requirements

- A **Windows** server (64-bit) or a **Linux** server with systemd and Python 3.9 or newer (Ubuntu, Debian, RHEL, Rocky
  and similar). The Windows installer includes everything it needs.
- A web browser. SiteMon serves its dashboard over HTTPS, on port 3443 by default.

## Install

Download the newest files from the [latest release](https://github.com/RowShack/SiteMon/releases/latest).

### Windows

1. Download `SiteMon-<version>.msi` and `SHA256SUMS.txt`.
2. [Check the download](#check-the-download).
3. Double-click the MSI and answer the questions:

   | Question | Choices |
   |---|---|
   | **Run SiteMon** | **As a Windows service** (recommended): starts with Windows, keeps monitoring when nobody is signed in and restarts itself if it stops. Or **as a program** that runs only while it is open. |
   | **Who can open the web interface** | **Other computers on the network** (adds a Windows Firewall rule) or **only this computer**. |
   | **Port** | 3443 by default (HTTPS). |

   At the end it opens SiteMon in your browser, where you create the first admin account.

Silent install (Intune, GPO, an RMM tool):

```
msiexec /i SiteMon-<version>.msi /qn MODE=service ACCESS=all PORT=3443
```

`MODE` is `service` or `app`, `ACCESS` is `all` or `local`. An upgrade keeps the earlier choices and all data: install the
newer MSI over the old one.

The service is called **SiteMon** in `services.msc`. Its data (settings, users, history, certificates, logs) is in
`C:\ProgramData\SiteMon`. Uninstalling keeps that folder; delete it yourself to remove everything.

**Windows may warn you.** The installer is not signed with a paid code-signing certificate yet, so SmartScreen can show
"Windows protected your PC". Check the SHA-256 below first. If it matches, choose **More info**, then **Run anyway**.

### Linux

```
sudo sh sitemon-<version>-linux.run
```

It asks whether to install a systemd service (recommended), the port, whether other computers may connect and, if ufw or
firewalld is active, whether to open the port. Run a newer `.run` over the top to upgrade; data is kept.

| | |
|---|---|
| Program | `/opt/sitemon` |
| Data | `/var/lib/sitemon` |
| Service | `systemctl status sitemon`; logs with `journalctl -u sitemon -f` |
| Command | `sudo sitemon --help` runs SiteMon's commands against the service's data |
| Uninstall | `sudo /opt/sitemon/uninstall.sh` (add `--purge` to delete the data too) |

For no questions: `sudo sh sitemon-<version>-linux.run --unattended --service --port 3443`. `--local-only`,
`--no-firewall` and `--no-service` also work; `--help` lists everything.

### Check the download

Each release lists `SHA256SUMS.txt`. The checksum of your file must match its line there.

```
# Windows (PowerShell)
Get-FileHash .\SiteMon-<version>.msi -Algorithm SHA256

# Linux
sha256sum -c SHA256SUMS.txt --ignore-missing
```

The in-app updater goes further: it only accepts releases signed with MonStack's own key, which is built into the app.

## First sign-in

Open `https://<server>:3443/`. The first time, SiteMon asks you to **create the first admin account**. If you create it
from another computer rather than on the server itself, it asks for a setup code, which is in `setup-code.txt` in the data
folder and in the log. SiteMon makes its own self-signed HTTPS certificate, so browsers warn the first time.

## Add an app by pulling

1. In the app (FortiMon, UniMon or MerakiMon) open **Settings > API keys** and make a key for **SiteMon (card)**. It is shown
   once.
2. In SiteMon open **Settings > Apps > + Add an app**. Enter the app's address (like `https://192.168.1.20:8443`) and the key,
   then **Test connection**.
3. SiteMon first shows the app's certificate fingerprint **without sending the key**. Compare it with the app's own
   (Settings > Web server there) and trust it. From then on SiteMon only talks to that exact certificate, and shows the app as
   offline if it changes, without sending the key.
4. SiteMon reads every app every 30 seconds. An app that cannot be reached keeps its last data on the screen, dimmed, with
   "last seen".

## Add an app by pushing

For an app SiteMon cannot reach:

1. In SiteMon open **Settings > Apps**, add an app in **push** mode and copy the **incoming key** it makes. It is shown once,
   kept hashed, tied to that one app, and can be replaced or revoked.
2. In the app open **Settings > API keys**, find **Send to SiteMon**, enter SiteMon's address and that key, set the same
   interval as in SiteMon (30 to 300 seconds) and save. **Send now** tests it.
3. If SiteMon has a self-signed certificate, use **Trust its certificate...** there and compare the fingerprint with
   SiteMon's own. Without it the app fails with a certificate error.
4. A push app is shown as offline when nothing arrives within 3 of its intervals.

## The wallboard

Open `https://<server>:3443/wall`. It shows a tile per site across all apps and refreshes every 30 seconds. Filters by app,
vendor and status are kept in the address (`#app=`, `#vendor=`, `#status=`), so you can bookmark one. For a screen nobody
signs in to, make a wallboard key under **Settings > API keys** and use its link.

## Try it without any apps

`sitemon --demo` (or `python sitemon.py --demo` from source) runs three stub apps, so the dashboard and wallboard have
something to show. It keeps its own settings and users.

## Next steps

Set up alerts for apps going offline and sites going critical, and users with roles: see the
[configuration reference](configuration.md).
