# Installing and Setting Up WinTAK, ATAK and iTAK — Beginner's Guide

How to install the three common TAK clients and get each one to a working basic state: **ATAK** (Android), **WinTAK** (Windows) and **iTAK** (iPhone/iPad). It covers install, first-run permissions, identity (callsign), map basics, and connecting to a TAK Server. For creating server accounts and the enrollment package, use the *User Profiles and Connecting Clients* guide. For offline maps, use the *MOBAC* guide.

> **Use at your own risk. No warranty.** This guide is an unofficial, community-written resource provided "as is." It is not affiliated with or endorsed by the TAK Product Center. Read the full **[Legal notice, license and disclaimer](#legal-notice-license-and-disclaimer)** at the end before you follow it. TAK clients can share your **live location** with everyone on a server. Know who is on your server before you connect.

---

## Placeholders used in this guide

| Placeholder | Meaning |
|---|---|
| `<SERVER_IP>` | Your TAK Server's address (a fixed address on your network) |
| `<tak_user>` / `<password>` | The account you were given on the server |
| `<server_name>` | The friendly name you want shown for the server |
| `<CALLSIGN>` | The name you appear as on the map |

Never post your real values, your server address or screenshots of your connection settings publicly.

---

## 1. Which client do I need?

| Client | Runs on | Notes |
|---|---|---|
| **ATAK-CIV** | Android phones and tablets | The most feature-rich client; supports plugins. The civilian edition is the one most people use |
| **WinTAK-CIV** | Windows PCs | Desktop view; good for planning and watching a map on a big screen |
| **iTAK** | iPhone and iPad | A simpler client with fewer features than ATAK; installed from the App Store |

All three can connect to the same TAK Server and see each other on the map, chat and share markers.

## 2. Before you start (all clients)

- [ ] Your TAK Server is running and you know it is reachable from your device.
- [ ] You have your **username and password** and the **enrollment data package** (`enrollmentDP.zip`) from the server's operator. (Or a QR code, if your server provides one.)
- [ ] The device is on the **same network** as the server for a LAN-only build, or on a VPN/overlay that your server's operator set up.
- [ ] The device's **date, time and time zone are set automatically.** A wrong clock breaks certificates and makes positions look stale.
- [ ] The device has enough free storage and battery.
- [ ] You understand what is shared: your location, callsign, chat messages and any markers you place.
- [ ] You downloaded software only from **official sources** (Section 3).

## 3. Where to get the software

| Client | Official source |
|---|---|
| ATAK-CIV | Google Play (search "ATAK-CIV"), or the downloads section at **tak.gov** (free account) |
| WinTAK-CIV | **tak.gov** downloads (free account) |
| iTAK | Apple **App Store** (search "iTAK") |

- **Avoid reposted APKs or installers** from forums and file-sharing sites. They can be outdated or tampered with.
- Do not redistribute TAK software. Each user should get it from the official source and accept the terms there.
- Versions change and the available builds may differ by region or account. Check the current release notes on tak.gov.

> **Matching versions matters.** The client and server should be compatible versions, and ATAK plugins must match the **exact ATAK version** installed. If you are unsure, ask your server operator which version to use.

---

## 4. ATAK (Android)

### 4.1 Install

**From Google Play:**
1. Open Google Play, search **ATAK-CIV**, and install it.
2. Check the publisher and name match the official listing.

**From tak.gov (APK, "sideloading"):**
1. Download the ATAK-CIV APK from tak.gov on the phone (or on a PC and copy it over).
2. Open the file. Android will ask you to allow installs from that source (your browser or file manager). Allow it **only for this install**, then turn it back off afterward.
3. Tap **Install**.

If you installed from one source and want the other, uninstall first. Mixing sources can fail because the app signatures differ. Back up any data you care about first.

### 4.2 First launch and permissions

1. Open ATAK. It may show a license or terms screen. Read and accept.
2. When Android asks for permissions, grant:
   - **Location** (set to allow all the time if you want background tracking)
   - **Storage / files** access (needed for maps and data packages)
   - **Notifications**
   - **Camera, microphone, Bluetooth** if you intend to use those features
3. ATAK may ask you to choose a storage location and a map type. The defaults are fine to start.
4. Android's **battery optimisation** can put ATAK to sleep. In Android settings, set ATAK's battery usage to **Unrestricted/Not optimised**.

### 4.3 Set your identity

1. Open the main menu (three lines / the "hamburger" icon), then **Settings**.
2. Open **Callsign and Device Preferences** (wording varies).
3. Set:
   - **Callsign:** a short name others will see. Keep it simple and non-identifying.
   - **Team color** and **Role:** how you appear to others.
   - **Device/UID:** leave as generated.

### 4.4 Learn the screen

| Area | What it does |
|---|---|
| Map | Shows you, others and markers |
| Top toolbar | Server status, GPS status, quick tools |
| Main menu (three lines) | Settings, Import Manager, tools, plugins |
| Bottom/side toolbars | Drawing, measuring, chat, contacts |
| Contact list | People on the server; tap one to centre, chat or navigate |
| Map layer button | Switch base map or imagery |

Useful basics:
- **Drop a marker:** tap and hold on the map, choose a marker type.
- **Measure:** use the range-and-bearing tool.
- **Center on yourself:** tap the location-lock icon.
- **Send a chat message:** open Contacts, pick a person or the "All Chat Rooms" channel.

### 4.5 Connect to the server

Choose one method.

**Method 1 (best): import the enrollment data package**
1. Copy `enrollmentDP.zip` to the phone.
2. In ATAK: **Settings/Menu → Import Manager → Import → Local SD** and select the zip.
3. When prompted, enter your **username and password**.
4. ATAK enrolls, gets its own certificate and connects.

**Method 2: Quick Connect or QR code** (if your server operator offers it)
1. Open the network connection settings (**Settings → Network Preferences → Network Connections**, or the Quick Connect option on the server screen).
2. Scan the QR code provided, then enter your credentials.

**Method 3: add the server by hand** (only if you have the address and trust certificate)
1. **Settings → Network Preferences → Network Connections → Add Server.**
2. Enter a description, `<SERVER_IP>`, port **8089** and protocol **SSL**.
3. Import the required certificates when asked, and enable the connection.

### 4.6 Check that it works

- The server indicator at the top should turn **green/connected**.
- Open Contacts and confirm other connected people appear.
- Ask someone to send you a chat message, and place a marker to see if they receive it.

### 4.7 Plugins (optional, later)

Plugins add features, but each must match your exact ATAK version and comes from a trusted source. Install them **after** the basic connection works, one at a time.

---

## 5. WinTAK (Windows)

### 5.1 System preparation

- A current **64-bit Windows 10 or 11** PC with enough RAM and free disk space. Check tak.gov for the stated requirements for your version.
- **Administrator rights** to install.
- Required runtimes. WinTAK usually needs a specific **.NET** runtime and the **Microsoft Visual C++ redistributables**. The installer may include or prompt for them; check the release notes for your version.
- Windows updates applied, and a working graphics driver (the map display relies on it).
- Antivirus will sometimes flag an installer. Download only from tak.gov, and add exclusions only if you are certain of the source.

### 5.2 Install

1. Log in at **tak.gov** and download **WinTAK-CIV** (and any required companion installers named in the release notes).
2. Right-click the installer and **Run as administrator**.
3. Follow the prompts and accept the defaults unless you have a reason not to.
4. Allow it through **Windows Firewall** when prompted (private networks at least).
5. Launch WinTAK from the Start menu.

### 5.3 First launch

1. Accept the terms. Choose a **callsign** and **team/role** when prompted, or set them later.
2. Choose a map source. A built-in base map is fine to start.
3. If prompted for a **location source**, choose your PC's location or set a manual position. Desktops without GPS will not report a true position unless you set one.

### 5.4 Connect to the server

1. Open **Settings** (gear icon), then the **Network / Network Connections** section.
2. Choose **Import** and select `enrollmentDP.zip`, or use the data package import feature.
3. When prompted, enter your **username and password**.
4. Confirm the connection shows as connected.

If you prefer manual setup, add a server with `<SERVER_IP>`, port **8089**, **SSL**, and import the certificates provided.

### 5.5 Basics

- **Chat and contacts:** open the contacts/chat panel to message people.
- **Markers and drawing:** use the toolbar to place points, routes and shapes.
- **Import Manager:** import maps, data packages and files.
- **Plugins:** add later, matching your WinTAK version.

---

## 6. iTAK (iPhone / iPad)

### 6.1 Install

1. Open the **App Store**, search for **iTAK**, and install it.
2. Check the publisher matches the official TAK listing.

### 6.2 First launch and permissions

1. Open iTAK. Accept the terms.
2. Allow **Location** (choose **Always** for background updates; while-in-use stops tracking when the screen is off).
3. Allow notifications if asked.
4. Turn on **Background App Refresh** for iTAK in iOS settings if you want it to keep reporting.

### 6.3 Connect to the server

**Method 1: data package**
1. Get `enrollmentDP.zip` onto the device (AirDrop, Files app, a private link).
2. In the **Files** app, tap the zip, then choose **Share / Open in iTAK** (iOS may unzip it first, in which case select the contents or share the original zip).
3. iTAK imports it and prompts for **username and password**.

**Method 2: QR code**
1. In iTAK, open the **Servers** area and choose to scan a QR code.
2. Scan the code provided by your server operator.
3. Enter your credentials when asked.

**Method 3: manual**
1. Open **Servers → Add Server**.
2. Enter a description, `<SERVER_IP>`, port **8089**, **SSL**.
3. Import the certificate or enable certificate enrollment as instructed by your operator.

### 6.4 Set your identity and check it works

- Open **Settings** and set your **callsign**, **team** and **role**.
- Confirm the server shows as connected, then confirm you can see and chat with others.

### 6.5 iTAK limits

iTAK is simpler than ATAK and does not support every feature or plugin. If you need a specific tool, check whether it exists in iTAK before relying on it.

---

## 7. Quick comparison

| Task | ATAK | WinTAK | iTAK |
|---|---|---|---|
| Install source | Google Play / tak.gov | tak.gov | App Store |
| Import server package | Import Manager | Settings / import | Open in iTAK / Servers |
| Plugins | Yes | Yes (different set) | Limited |
| Position source | Phone GPS | Manual or device | Phone GPS |
| Best for | Field use | Planning/monitoring | Simple field use on Apple |

## 8. Final checklist (per device)

- [ ] Installed from an official source
- [ ] Date and time set automatically
- [ ] Permissions granted (location, storage/files, notifications)
- [ ] Battery optimisation / background refresh handled
- [ ] Callsign, team and role set
- [ ] Server connected (indicator green)
- [ ] Can see other users and they can see you
- [ ] Chat and marker test passed
- [ ] Reboot test: restart the device and confirm it reconnects
- [ ] You know how to turn off location sharing or disconnect quickly
- [ ] You do not rely on this as your only navigation or communication method

---

## Common mistakes and corrections

| Mistake | What happens | Correction |
|---|---|---|
| Installing from random APK or installer sites | Outdated or tampered software | Use Google Play, tak.gov or the App Store only |
| Mixing Google Play and sideloaded ATAK | Install or update fails | Uninstall one source before installing the other |
| Leaving "install unknown apps" permanently enabled | Weaker phone security | Turn it off again after installing |
| Granting location "only while using" | Position stops when screen is off | Allow "all the time" (Android) or "Always" (iOS) |
| Not turning off battery optimisation | Connection drops in the background | Set ATAK to unrestricted |
| Wrong phone clock or time zone | Certificate or stale-position problems | Turn on automatic date and time |
| Importing the wrong zip (not `enrollmentDP.zip`) | No sign-in prompt | Use the enrollment package |
| Typing the server address wrong in a manual setup | Never connects | Prefer the data package; check each digit |
| Device on the wrong network | Server unreachable | Same network, or the VPN your operator provided |
| Server IP changed | Old package stops working | Ask the operator for a new package |
| Installing plugins before the basics work | Hard to find the cause of problems | Get connected first, add one plugin at a time |
| Plugin version doesn't match ATAK | Plugin won't load or crashes ATAK | Match the exact ATAK version |
| Missing WinTAK runtimes | Won't start or crashes | Install the required .NET and VC++ runtimes |
| Running WinTAK installer without admin rights | Incomplete install | Run as administrator |
| Blocking WinTAK in Windows Firewall | Can't connect | Allow it on private networks |
| Using a desktop with no GPS and expecting a position | Appears at the wrong place | Set a manual position |
| Using a plain Files-app unzip on iOS and losing the package | iTAK doesn't import it | Share the original zip to iTAK |
| Using one shared login on many devices | Users can't be told apart | One account per person |
| Using a nickname that reveals your real identity or sensitive details | Others can identify you | Use a neutral callsign |
| Posting screenshots of settings | Exposes your server | Blur or crop addresses and credentials |
| Treating the app as your only navigation tool | Failure leaves you stranded | Carry independent backups |

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| App installs but crashes on launch | Version mismatch, missing runtime (WinTAK), or low storage. Reinstall from the official source; check requirements |
| No sign-in prompt after import | Wrong package, or enrollment isn't enabled on the server. Ask the operator |
| Sign-in rejected | Wrong username or password. Ask the operator to reset it |
| Stuck on "connecting" | Server unreachable, wrong address, blocked port or wrong network. Check the network first |
| Connected but see no one | Other users aren't online, or you're in different groups |
| Position doesn't update | Location permission set too low, GPS off, or battery restrictions |
| Position wrong on WinTAK | No GPS source; set one manually |
| Connects, then drops | Weak Wi-Fi, battery optimisation or a wrong clock |
| Plugin missing after an update | The plugin must match the new ATAK version; reinstall the matching one |
| Map is blank | No internet and no offline map. Load an offline map (see the MOBAC guide) |
| iTAK won't open the zip | Use Share / Open in iTAK from the original zip |

---

## Credits and links

- TAK Product Center (ATAK, WinTAK, iTAK, TAK Server): https://tak.gov
- Apple App Store and Google Play for the official client listings

*Client versions, menu wording and requirements change often. Verify against the current official documentation and release notes before relying on any step here.*

---

## Legal notice, license and disclaimer

**This is general boilerplate, not legal advice.** If you need certainty about your obligations, ask a qualified lawyer.

### Use at your own risk; no warranty

This guide, and any scripts, configuration files or other code distributed with it (together, the "Materials"), are provided **"as is" and "as available," without warranty of any kind**, express or implied. That includes, without limitation, any warranty of merchantability, fitness for a particular purpose, accuracy, completeness, security, reliability or non-infringement.

To the fullest extent permitted by law, the authors and contributors are **not liable** for any claim, damages or other liability, whether in contract, tort or otherwise, arising from or connected with the Materials or their use. This includes data loss, device malfunction, downtime, security incidents, exposure of data or locations, financial loss, injury, or any other harm. You follow this guide entirely at your own risk.

### Accuracy and AI assistance

Parts of this guide were drafted with the help of an AI assistant and may contain errors, omissions or outdated steps. Software, download locations, system requirements and menu names change between versions and platforms. Always check against the official documentation for ATAK, WinTAK, iTAK, Android, iOS and Windows, and test on non-critical devices first.

### Commands and installs can be destructive

Installing software, changing device permissions, uninstalling apps and sideloading files can cause data loss or weaken device security. Back up anything you care about first, and only install from sources you trust.

### Not affiliated; trademarks

This is an unofficial, independent community guide. It is **not affiliated with, endorsed by or sponsored by** the TAK Product Center, any government agency, Apple, Google, Microsoft, or any other company or project mentioned. TAK, ATAK, WinTAK, iTAK, TAK Server, Android, iOS, iPhone, iPad, Windows, Google Play, App Store and all other names, logos and trademarks belong to their respective owners and are used here only to identify the products.

### Third-party software and content

- **TAK Server and TAK clients** are not included. Obtain them from the official source (tak.gov, Google Play or the App Store) and follow the terms you accept there. Do not redistribute them.
- **Plugins and third-party tools** have their own licenses, maintainers and risks. You are responsible for reviewing and complying with them.
- Links point to external sites that the authors do not control.

### Security

This guide covers **basic client setup**, not a hardened deployment. It makes no promise that any resulting setup is secure. Keep devices updated, protect enrollment packages, certificates and passwords, and do not connect to servers you do not trust.

### Privacy, consent and the law

TAK clients can transmit your **live location and other data** to a server and to everyone on it. You are responsible for complying with all laws and rules that apply to you, including privacy and data-protection law, consent from the people whose data is shared, radio and telecommunications rules, employer or organisational policies, and export-control and other legal requirements for the software and hardware you use. Do not use these Materials for any unlawful purpose, including to track anyone without their knowledge and consent.

### Not for safety-critical use

Do not rely on a phone, a computer or any software described here as your **only** means of navigation, communication, tracking or emergency response. Equipment fails, batteries die, networks drop and data can be wrong or stale. Carry independent backups, test your setup in advance, and use sound judgment, especially outdoors or in any situation where a failure could put someone at risk.

### License

Unless stated otherwise, and subject to the disclaimers above:

- **The text of this guide** is licensed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license.
- **Any code, scripts or configuration files** distributed with this guide are licensed under the **MIT License**.

Copyright (c) 2026 4head-tac-tech. Third-party software and content keep their own licenses. By contributing, you agree that your contribution is licensed on the same terms. The warranty disclaimer and limitation of liability above apply to everyone who uses, copies, modifies or redistributes the Materials.

### Reporting problems

Found a mistake, a security issue or an outdated step? Please open an issue or a pull request in the project repository: https://github.com/4head-tac-tech/TAK-Field-Kit.
