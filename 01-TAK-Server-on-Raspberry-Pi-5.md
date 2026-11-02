# TAK Server on a Raspberry Pi 5 (Raspberry Pi OS Bookworm) — Setup Guide

A start-to-finish walkthrough for running an official **TAK Server 5.x** on a **Raspberry Pi 5** and connecting **ATAK** on an Android phone. Written from a real first-time build, including the mistakes that cost time.

> **Use at your own risk. No warranty.** This guide is an unofficial, community-written resource provided "as is." It is not affiliated with or endorsed by the TAK Product Center. It uses a third-party community installer that runs as root. Read scripts before you run them. Do not redistribute TAK Server files (get them from tak.gov yourself). Do not expose this server to the public internet without hardening it first. See the full **[Legal notice, license and disclaimer](#legal-notice-license-and-disclaimer)** at the end of this guide before you follow it.

---

## What you need

| Item | Notes |
|---|---|
| Raspberry Pi 5 (8 GB recommended) | 4 GB may work, 8 GB is comfortable |
| microSD card, 64–128 GB, A2 / high-endurance | Fine for a first build; consider an SSD later |
| Official Pi 5 USB-C power supply (27 W) | Under-powering causes random instability |
| Wi-Fi or Ethernet | Ethernet is more reliable; Wi-Fi works |
| A Windows/Mac/Linux PC | For flashing and SSH |
| A free **tak.gov** account | Needed to download TAK Server |
| An Android phone with **ATAK-CIV** | To test the connection |
| Firefox (on the PC) | Easiest for importing the admin client certificate |

---

## 1. Flash the OS

> **Heads up:** writing an image **erases everything** on the target card. Double-check that you selected the right drive before you click Write.

1. Install **Raspberry Pi Imager** on your PC.
2. Choose **Raspberry Pi 5** as the device.
3. Choose **Raspberry Pi OS (64-bit)** based on **Debian 12 "Bookworm"**.
   - It may be under *Raspberry Pi OS (other)* or a *Legacy* entry depending on the Imager version. Verify that it says Debian 12 / Bookworm.
   - **Do not use Trixie (Debian 13).** It does not ship Java 17 or the matching PostgreSQL packages that TAK Server's installer expects, and the install fails.
4. Click the settings/customisation (gear / "Edit Settings") and set:
   - Hostname: `<hostname>`
   - Username and a strong password: `<username>` / `<password>`
   - Wi-Fi SSID and password (if not using Ethernet) and your country
   - **Enable SSH** (password authentication is fine for a LAN-only build)
5. Write to the SD card. Re-flashing wipes it, so there is no need to format it first.

## 2. First boot and SSH

1. Insert the card, plug in power, and wait 2–3 minutes.
2. From your PC, open a terminal (PowerShell or Command Prompt on Windows) and run:
   ```
   ssh <username>@<hostname>.local
   ```
   If `.local` does not resolve, find the Pi's IP in your router's device list and use `ssh <username>@<PI_IP>`.
3. Type `yes` to accept the host fingerprint on first connect.
4. If you re-flash later and get a "host key changed" warning, clear the old key on your PC:
   ```
   ssh-keygen -R <hostname>.local
   ```
5. Confirm the OS version:
   ```
   cat /etc/os-release
   ```
   You should see `VERSION_CODENAME=bookworm`.
6. Update:
   ```
   sudo apt update && sudo apt full-upgrade -y
   ```

## 3. Download TAK Server

1. Log in at **tak.gov** and open the downloads section.
2. Download these two files for the Debian/Raspberry Pi build (version numbers will differ):
   - `takserver_<version>_all.deb`
   - `deb_policy.pol`
3. Copy them to the Pi. **Run this on your PC, not in the SSH window:**
   ```
   scp takserver_<version>_all.deb deb_policy.pol <username>@<PI_IP>:~/
   ```
   Notes:
   - The destination must end with `:~/` (or a path), or `scp` just copies locally.
   - Username is case-sensitive.
   - Filenames must match exactly. Check with `ls -lh` on the Pi afterwards.

TAK Server is distributed by the TAK Product Center under its own terms. You are responsible for reading and following those terms, including any limits on redistribution and export.

## 4. Get the community installer

[`myTeckNet/installTAK`](https://github.com/myTeckNet/installTAK) automates the Java/PostgreSQL install, certificate generation and data-package creation. It is a third-party project with its own license and maintainers. This guide does not control or vouch for its contents, and it runs with root privileges, so review it before you run it.

On the Pi:
```
sudo apt install -y git
git clone https://github.com/myTeckNet/installTAK.git --recurse-submodules
cd installTAK
cp ~/takserver_<version>_all.deb ~/deb_policy.pol .
chmod +x installTAK
```
Make sure there is a space before `--recurse-submodules`, and that you cloned the up-to-date repository rather than an outdated fork.

## 5. Run the installer

```
sudo ./installTAK takserver_<version>_all.deb
```

A text-based (whiptail) wizard appears. Navigation: **arrow keys** to move, **spacebar** to tick a box, **Enter** to confirm, **Tab** to jump to buttons.

Recommended answers for a LAN-only first build:

| Prompt | Answer |
|---|---|
| Root CA / Intermediate (issuing) CA names | Anything descriptive, e.g. `MyRootCA`, `MyIssuingCA` |
| Certificate details (country, state, city, org) | Your choice; this is not sent anywhere |
| Certificate password | Choose and **record it**. Don't use the default |
| Certificate Auto-Enrollment | **Yes** |
| Web options | **Tick** `enableAdminUI` and `enableWebtak` with the **spacebar**; leave `enableNonAdminUI` off |
| Federation | Disabled (revisit later) |
| FIPS | False |
| Connection type | SSL |
| Use an FQDN? | **No**, then enter the Pi's IP address |

> **Common mistake:** pressing Enter on the web-options screen without ticking boxes leaves everything unchecked and the admin UI won't be enabled. If that happens, use the wizard's **Reset** and tick the boxes with the spacebar.

The install takes several minutes on a Pi. When it finishes it creates an admin client certificate (`webadmin.p12`) and enables the `ufw` firewall (SSH stays allowed).

## 6. Verify the server

```
sudo systemctl status takserver
```
Expect `active (running)`. (Command names are case-sensitive: `systemctl`, not `Systemctl`.)

Check the listening ports:
```
sudo ss -tlnp | grep -E '8089|8443|8446'
```

| Port | Purpose |
|---|---|
| 8089 | Client (ATAK) SSL connections |
| 8443 | Admin UI (requires client certificate) |
| 8446 | Password-based WebTAK / certificate enrollment |

The first startup can take a few minutes while it initialises the database, so wait before assuming something is broken.

## 7. Admin access (client certificate)

1. Copy the admin certificate to your PC (run on the PC):
   ```
   scp <username>@<PI_IP>:/opt/tak/certs/files/webadmin.p12 .
   ```
   (Location may vary by installer version; `sudo find / -name webadmin.p12 2>/dev/null` will find it. If permission is denied, copy it to your home directory with `sudo cp` and `sudo chown` first.)
2. In **Firefox**: *Settings → Privacy & Security → View Certificates → **Your Certificates** tab → Import*.
   - Change the file-type dropdown to **All Files** or the `.p12` won't be listed.
   - Do **not** use the *Authorities* tab.
3. Enter the certificate password you chose in step 5.
4. Browse to `https://<PI_IP>:8443` and select the certificate. Accept the self-signed warning.
5. A good health check is the metrics page: `https://<PI_IP>:8443/Marti/metrics/index.html`.

## 8. Create a user and connect ATAK

### Create a user
```
sudo java -jar /opt/tak/utils/UserManager.jar usermod -a -p '<password>' <user>
```
- The file name is `UserManager.jar` with a capital **U**.
- Password rule: **15+ characters** with upper case, lower case, a digit and a special character.
- This creates a normal (non-admin) user.

### Find the connection package
The installer generated enrollment data packages. Find the recent ones:
```
sudo find / -name "*.zip" -mmin -120 -not -path "/proc/*" 2>/dev/null | head
```
Use `enrollmentDP.zip` (you don't need the QUIC variant). Copy it somewhere you can reach, then to your PC:
```
scp <username>@<PI_IP>:~/enrollmentDP.zip .
```

Verify it points at the right address:
```
unzip -p ~/enrollmentDP.zip config.pref | grep -i connect
```
It should show your `<PI_IP>` with port `8089`.

### Import into ATAK
1. Transfer `enrollmentDP.zip` to the phone (USB, cloud drive, etc.).
2. In ATAK: **Settings/Menu → Import Manager → Import → Local SD** and choose the zip.
3. When prompted, sign in with the username/password you created. The certificate is enrolled automatically.
4. Confirm the server shows as connected in ATAK and **Clients Connected** rises on the server metrics page.

Your phone must be on the same network as the Pi for a LAN-only setup.

## 9. Give the Pi a fixed IP

The ATAK package points at a specific address, so the Pi's IP must not change. Pick one:

**Option A (preferred): DHCP reservation on your router.** Reserve the Pi's address by MAC in your router's admin page. Router credentials are the admin ones, which may differ from the Wi-Fi password printed on the sticker.

**Option B: static IP on the Pi with NetworkManager.**
```
nmcli con show                       # find the connection name, e.g. "preconfigured"
sudo nmcli con mod <connection> \
  ipv4.method manual \
  ipv4.addresses <PI_IP>/24 \
  ipv4.gateway <router-ip> \
  ipv4.dns "<router-ip>,1.1.1.1"
```
- `ipv4.addresses` must be set in the same command as (or before) the gateway.
- Double-check every digit. A typo here can make the Pi unreachable.
- Verify before rebooting:
  ```
  nmcli -g ipv4.method,ipv4.addresses,ipv4.gateway con show <connection>
  ```
- Then `sudo reboot` and SSH back in on the new address.

Choose an address outside your router's DHCP pool to avoid conflicts.

## 10. Housekeeping

- **Reboot test.** `sudo reboot`, wait, and confirm ATAK reconnects without touching the server.
- **Back up the SD card** once it works (e.g. Win32 Disk Imager on Windows) so you can restore in minutes.
- **Passwords.** Replace any placeholder or default passwords, and don't post screenshots with passwords or certificate passphrases visible.
- **Keep it on the LAN** until you have decided on a proper remote-access approach (VPN/mesh overlay, firewall rules). Don't just forward ports.
- **Keep a record** of your certificate password, user list and IP in a password manager.

---

## Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| Installer fails, "Java 17 not found" | You're on Trixie. Re-flash Bookworm 64-bit |
| `scp` says permission denied / does nothing | Run from the PC window, not the Pi; check username case; include `:~/` destination |
| `No such file` on `.deb` | Filename incomplete; copy exact name from `ls -lh` |
| `command not found` for obvious commands | Check case and spaces (`sudo`, `systemctl`, `UserManager.jar`) |
| Admin UI (8443) won't load | Web options weren't ticked in the wizard, or the cert isn't imported in the right Firefox tab |
| Firefox can't see the `.p12` | File-type filter: switch to *All Files* |
| User creation rejected | Password under 15 chars or missing a character class |
| ATAK can't connect | Wrong IP in package, phone on a different network, IP changed (set a fixed IP), or server still starting |
| Lost SSH after IP change | Typo in static config; reconnect via monitor/keyboard, or re-flash and redo step 9 carefully |
| Host key changed warning | `ssh-keygen -R <host>` after re-flashing |

---

## Credits and links

- TAK Server and ATAK: TAK Product Center — https://tak.gov
- Community installer: https://github.com/myTeckNet/installTAK
- Raspberry Pi Imager: https://www.raspberrypi.com/software/

*Tested with TAK Server 5.8 on Raspberry Pi OS 64-bit (Debian 12 Bookworm), Raspberry Pi 5 8 GB.*

---

## Legal notice, license and disclaimer

**This is general boilerplate, not legal advice.** If you need certainty about your obligations, ask a qualified lawyer.

### Use at your own risk; no warranty

This guide, and any scripts, configuration files or other code distributed with it (together, the "Materials"), are provided **"as is" and "as available," without warranty of any kind**, express or implied. That includes, without limitation, any warranty of merchantability, fitness for a particular purpose, accuracy, completeness, security, reliability or non-infringement.

To the fullest extent permitted by law, the authors and contributors are **not liable** for any claim, damages or other liability, whether in contract, tort or otherwise, arising from or connected with the Materials or their use. This includes data loss, corrupted or wiped storage, hardware damage, downtime, security incidents, exposure of data or locations, financial loss, injury, or any other harm. You follow this guide entirely at your own risk.

### Accuracy and AI assistance

Parts of this guide were drafted with the help of an AI assistant and then tested on one hardware and software combination (see the version note above). It may contain errors, omissions or outdated steps. Software, download locations and menu names change. Always check against the official documentation for TAK Server, Raspberry Pi OS and any other software you use, and test on non-critical hardware first.

### Commands can be destructive

Several steps use `sudo`, overwrite storage, change network settings or run third-party scripts as root. A mistake can erase data or lock you out of the device. Back up anything you care about first, and read every command and script before you run it.

### Not affiliated; trademarks

This is an unofficial, independent community guide. It is **not affiliated with, endorsed by or sponsored by** the TAK Product Center, any government agency, Raspberry Pi Ltd, or any other company or project mentioned. TAK, ATAK, WinTAK, iTAK, TAK Server, Raspberry Pi, Android, Firefox and all other names, logos and trademarks belong to their respective owners and are used here only to identify the products.

### Third-party software and content

- **TAK Server and TAK clients** are not included. Obtain them from the official source (tak.gov) and follow the terms you accept there. Do not redistribute them.
- **Third-party tools** (for example installTAK, Java, PostgreSQL, Raspberry Pi Imager) have their own licenses, maintainers and risks. You are responsible for reviewing and complying with them. This guide does not audit or guarantee them.
- Links point to external sites that the authors do not control.

### Security

This guide covers a **LAN-only first build**, not a hardened production deployment. It makes no promise that the resulting server is secure. Do not expose the server to the public internet without proper hardening, and keep certificates, passwords and backups protected. You are responsible for securing your systems and the data on them.

### Privacy, consent and the law

A TAK Server can collect and share the **live location and other data** of everyone connected to it. You are responsible for complying with all laws and rules that apply to you, including privacy and data-protection law, consent from the people who use your server, radio and telecommunications rules, and export-control and other legal requirements for the software and hardware you use. Do not use these Materials for any unlawful purpose.

### Not for safety-critical use

Do not rely on a home-built server, a phone or any software described here as your **only** means of navigation, communication, tracking or emergency response. Equipment fails, batteries die, networks drop and data can be wrong or stale. Carry independent backups, test your setup in advance, and use sound judgment, especially outdoors or in any situation where a failure could put someone at risk.

### License

Unless stated otherwise, and subject to the disclaimers above:

- **The text of this guide** is licensed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license.
- **Any code, scripts or configuration files** distributed with this guide are licensed under the **MIT License**.

Copyright (c) 2026 4head-tac-tech. Third-party software and content keep their own licenses. By contributing, you agree that your contribution is licensed on the same terms. The warranty disclaimer and limitation of liability above apply to everyone who uses, copies, modifies or redistributes the Materials.

### Reporting problems

Found a mistake, a security issue or an outdated step? Please open an issue or a pull request in the project repository: https://github.com/4head-tac-tech/TAK-Field-Kit.
