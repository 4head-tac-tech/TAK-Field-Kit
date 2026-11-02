# Adding Meshtastic to WinTAK, ATAK and iTAK — Guide

How to link a **Meshtastic** LoRa radio mesh to the three common TAK clients so positions (PLI) and chat can travel over radio when there is no cell, Wi-Fi or internet. Order in this guide: **WinTAK first**, then **ATAK**, then **iTAK**. Each client uses a different method, so do not mix the steps between sections.

> **Use at your own risk. No warranty.** This guide is an unofficial, community-written resource provided "as is." It is not affiliated with or endorsed by the TAK Product Center or the Meshtastic project. Read the full **[Legal notice, license and disclaimer](#legal-notice-license-and-disclaimer)** at the end before you follow it. A radio mesh broadcasts your **live location** to every node in range that shares your channel. Know who is on your channel.

**ATAK has two options** (Section 4): the built-in TAK Server in the current Meshtastic Android app, or the ATAK plugin with a matched app version.

**Assumes:** you have already installed your TAK clients (see the *Install and Basic Setup* guide) and, if you use one, a working TAK Server (see the *TAK Server* guides). Meshtastic works without a TAK Server.

---

## Placeholders used in this guide

| Placeholder | Meaning |
|---|---|
| `<NODE_IP>` | The Meshtastic node's address if it is on Wi-Fi/LAN |
| `<COM_PORT>` | The serial port a USB node appears as (e.g. `COM5`) |
| `<CHANNEL_NAME>` / `<PSK>` | Your private Meshtastic channel name and key |
| `<CALLSIGN>` | The name you appear as on the TAK map |
| `<SERVER_IP>` | Your TAK Server's address, if you use one |

Never publish your real channel key, addresses or screenshots of your channel QR code.

---

## Versions used (a starting point)

These are the versions this guide was put together with. Use them as a **place to start**, not a guarantee. Other versions may work, and newer ones may change menus or behaviour.

| Item | Version used | Notes |
|---|---|---|
| **ATAK-CIV** | **5.6.0.12**, Google Play build (Plugin API `5.6.0.CIV`) | Shown under ATAK → Settings → About |
| **Android** | **16** | The plugin release notes also list Android 16 as tested |
| **Meshtastic ATAK Plugin** | **1.1.42**, the **`5.6.0-civ-release`** build | The same release also offers a `5.4.0` build, which does **not** match ATAK 5.6. Match the file to your own ATAK version |
| **Meshtastic Android app** | The **current Google Play version** at the time, which includes the built-in **TAK Server** screen (Option A) | Exact version number not recorded. Check Settings → About in your app |
| **Meshtastic Android app for the plugin (Option B)** | **2.7.x** (the plugin 1.1.42 release notes name **2.7.13** as tested) | The plugin **bound and worked** with a 2.7 version of the app. The exact build number is not recorded, so use 2.7.13 as your starting point |
| **TAK Server** | **5.8** on Raspberry Pi OS 64-bit (Debian 12 "Bookworm"), Raspberry Pi 5 | From the *TAK Server on a Raspberry Pi 5* guide |

**What has and has not been confirmed:**
- **Confirmed:** ATAK 5.6.0.12 accepted and loaded the plugin 1.1.42 `5.6.0` build. The built-in TAK Server screen in the current Meshtastic app turns on and shows "waiting for ATAK". ATAK connected to the TAK Server and showed other clients from it.
- **Confirmed (Option B):** with the **current** Meshtastic app the plugin showed **"Failed to bind to Meshtastic IMeshService"** and a red icon (Section 4.7). After switching to a **2.7 version** of the Meshtastic app, the plugin worked. So on ATAK 5.6.0.12, the plugin needs the older app, not the newest one.
- **Confirmed (WinTAK):** a chat message sent from WinTAK through the TAK Meshtastic Gateway was delivered to an ATAK device over the mesh. A test message sent from a phone on the mesh was also received in WinTAK, so chat works in both directions. See the companion *Meshtastic Node to WinTAK Gateway (Windows)* guide.
- **Not confirmed:** whether mesh data reaches a TAK Server through either option (relay to server), and how Option A (built-in TAK Server) behaves end to end. Test both directions yourself.
- Plugin releases are tested by their authors on a specific ATAK, Android and Meshtastic app combination (the 1.1.42 notes list ATAK 5.6.0.13). Always read the notes of the release you download.

**If your versions differ:** find your ATAK **Plugin API** line first, then pick the plugin APK whose filename contains the same version. Everything else follows from that choice.

---

## 1. How it fits together (read this first)

| Piece | What it is |
|---|---|
| **Meshtastic node** | A LoRa radio running Meshtastic firmware. Carries messages over radio from node to node |
| **TAK client** | WinTAK, ATAK or iTAK. Shows the map, chat and markers |
| **Bridge** | Software that turns TAK data (Cursor on Target, "CoT") into Meshtastic packets and back |
| **Channel** | The shared name and key that nodes must have in common to hear each other |

**The bridge is different per client:**

| Client | Bridge | Radio connection |
|---|---|---|
| **WinTAK** | **TAK Meshtastic Gateway** (a small community program running on the same PC) | USB serial or Wi-Fi/TCP |
| **ATAK** | **Option A:** the TAK Server built into the current Meshtastic Android app. **Option B:** the **Meshtastic ATAK Plugin** plus a matching Meshtastic Android app | Bluetooth (via the Meshtastic app) |
| **iTAK** | **TAK Server built into the Meshtastic iOS app** (iTAK connects to it on the same device) | Bluetooth (via the Meshtastic app) |

**Important limits (all clients):**
- LoRa is **slow and low-bandwidth**. Expect position updates, short chat and small markers. Do **not** expect photos, files, data packages, maps or video over the mesh.
- Features differ by client and version. Not every CoT type crosses the mesh.
- A phone or PC with a node is a **gateway** between the two worlds. Only things that pass through a bridge cross over.

## 2. Before you start (all clients)

- [ ] Your TAK client already works on its own (position shows, chat works).
- [ ] You have **at least two** Meshtastic nodes so you can test a real link.
- [ ] Nodes run **current, matching** Meshtastic firmware. Check the Meshtastic docs for the version your bridge requires.
- [ ] Nodes are set to the **correct radio region** for your location (a legal requirement, not an option).
- [ ] You created a **private channel** with your own name and a fresh random key, not the default public one, and loaded the same channel on every node. Share the key privately.
- [ ] You decided who will have the key. Everyone with it can see positions sent on the channel.
- [ ] Devices have **automatic date and time** set.
- [ ] You know the location-sharing risk (Section 6) and everyone agreed.

### Prepare each Meshtastic node (once)

1. Flash current firmware and set the **region** (use the Meshtastic app or web flasher from the official site).
2. Set a long name and short name that are **not personally identifying**.
3. Set the node **role** for the job:
   - **TAK**: a node paired to a TAK device that sends and receives TAK data.
   - **TAK Tracker**: a standalone node that broadcasts its own position to TAK maps with no phone attached.
4. Create or import your private channel (same name, same key, same modem preset on every node).
5. Turn on GPS/position on nodes that should report a location.
6. Check the TAK settings on the node (in the Meshtastic app: **Settings → Module Config → TAK**, wording varies by version).

> Verify the role names and menu paths against the current Meshtastic documentation. They change between app and firmware versions.

---

## 3. WinTAK (Windows) — start here

WinTAK has **no official Meshtastic plugin**. The usual approach is the community **TAK Meshtastic Gateway**, a small program on the same PC that listens for WinTAK's network traffic and passes chat and position messages to a Meshtastic node, and the other way around.

**What it carries:** chat and position (PLI) messages. Do not expect everything WinTAK can do to cross the mesh.

### 3.1 What you need

| Item | Notes |
|---|---|
| Windows PC with WinTAK working | Install WinTAK first |
| A Meshtastic node | Connected by **USB**, or reachable on your **LAN** over Wi-Fi |
| **Python 3.12** (64-bit) | The gateway's Windows instructions specify 3.12 |
| Internet during install | Needed once to download the Python packages |
| Admin rights | For installing Python and USB drivers |

### 3.2 Connect the node

**USB (simplest):**
1. Plug the node into the PC with a **data** USB cable (charge-only cables do not work).
2. Open **Device Manager → Ports (COM & LPT)** and note the port, e.g. `<COM_PORT>`.
3. If no port appears, install the USB driver for your node's chip (check your board's documentation).
4. **Close every other program** that might hold the port open, such as the Meshtastic web client or a serial monitor. Only one program can use the port at a time.

**Wi-Fi / LAN (alternative):**
1. Enable Wi-Fi on the node and join it to your network (Meshtastic app → network settings).
2. Give the node a fixed address or DHCP reservation and note it as `<NODE_IP>`.
3. The PC must be on the same network.

### 3.3 Install the gateway

Open **PowerShell**, then run these one at a time. (Use a virtual environment so nothing else on the PC is affected.)

```
python -m venv tak_meshtastic_gateway_venv
.\tak_meshtastic_gateway_venv\Scripts\activate
```

Then follow the **current install instructions on the project's page** (search "TAK Meshtastic Gateway" on pypi.org). At the time of writing they include a special prebuilt package for Windows plus these two commands:

```
pip install git+https://github.com/snstac/takproto@refs/pull/16/merge
pip install tak-meshtastic-gateway
```

> **Read before you run.** This is a third-party project with its own license and maintainers, installed from the internet. This guide does not audit it. The exact install steps change, so trust the project's own page over this guide.

### 3.4 Run it

With the virtual environment still active:

| Node connection | Command |
|---|---|
| USB | `tak_meshtastic_gateway` |
| Wi-Fi / LAN | `tak_meshtastic_gateway --mesh-ip <NODE_IP>` |

Run `tak_meshtastic_gateway --help` to see all options for your version. Keep this window open while you use WinTAK; closing it stops the bridge.

Common options listed by the project include the IP of the machine running the TAK client (default is the local PC) and a TCP port for direct messages (default 4243).

### 3.5 Set up WinTAK to talk to it

The gateway listens for WinTAK's **multicast** network traffic, so WinTAK must be sending it.

1. In WinTAK open **Settings → Network (or Network Preferences)**.
2. Make sure **multicast / SA (situational awareness) broadcast** is **on** for sending and receiving. Wording varies by version.
3. Make sure Windows Firewall allows WinTAK and Python on **private networks**.
4. Set your **callsign, team and role** in WinTAK.
5. If the PC has no GPS, set a **manual position** (otherwise nothing sensible is shared).

### 3.6 Test

1. Start the gateway, then WinTAK.
2. Have a second Meshtastic node with a phone (running the Meshtastic app) within radio range on the **same channel**.
3. In WinTAK send a short chat message. It should appear on the other node's Meshtastic app.
4. Send a message from the Meshtastic app. It should appear in WinTAK.
5. Check that the node's position shows up on the WinTAK map.

### 3.7 WinTAK mistakes and fixes

| Mistake | What happens | Correction |
|---|---|---|
| Charge-only USB cable | No COM port | Use a data cable |
| Another program holds the COM port | Gateway cannot open the port | Close other serial programs |
| Wrong Python version | Package install fails | Use 64-bit Python 3.12 as the project states |
| Installing into the system Python | Package conflicts | Use a virtual environment |
| Closing the gateway window | Bridge stops | Leave it running |
| Multicast turned off in WinTAK | Gateway hears nothing | Turn on SA multicast in network settings |
| Windows Firewall blocking | Nothing crosses | Allow WinTAK and Python on private networks |
| Different channel key/name on nodes | Nodes cannot hear each other | Same channel on all nodes |
| PC with no GPS | You appear at the wrong place | Set a manual position |
| Expecting files/maps over the mesh | Nothing arrives | LoRa carries small text and position only |
| Node on Wi-Fi, address changed | Gateway cannot find it | Reserve a fixed address |

---

## 4. ATAK (Android)

There are **two ways** to connect ATAK to Meshtastic. Both go through the **Meshtastic Android app**, which holds the Bluetooth link to your radio. Pick **one**. Do not run both at the same time.

| | **Option A: built-in TAK Server** | **Option B: ATAK plugin** |
|---|---|---|
| How it works | The Meshtastic app runs a small TAK server on your phone. ATAK connects to it like any other server | A plugin inside ATAK sends and receives TAK data through the Meshtastic app |
| Meshtastic app version | Current (needs the TAK Server screen) | The version the plugin release was tested with, often an older one |
| Extra download | None (just a data package exported from the app) | A plugin APK that **matches your exact ATAK version** |
| Best when | You want the simplest setup on current software | No plugin exists for your ATAK, or you need the plugin's own settings such as "relay to server" |
| Main risk | Newer feature; behaviour may change between app versions | Plugin and app version mismatches ("failed to bind") |

**Which should I start with?** Start with **Option A**. It needs no plugin and no downgrade. Use **Option B** if Option A does not do what you need.

### 4.1 What you need (both options)

| Item | Notes |
|---|---|
| Android phone with ATAK-CIV working | Note the **exact ATAK version** (ATAK → Settings → About) |
| **Meshtastic Android app** | Google Play for Option A. See Option B for version notes |
| A Meshtastic node with Bluetooth | Role set to **TAK** (Section 2) |
| Your private channel loaded on the node | Same name, key and preset as your other nodes |

### 4.2 Option A: the built-in TAK Server (current Meshtastic app)

**What the TAK Server screen contains** (Meshtastic app → **Settings → TAK Server**; wording and layout may differ by version):

| Setting | What it does |
|---|---|
| **Enable Local TAK Server** | Starts a local TLS server on port 8089 for ATAK. It listens on the phone itself (`127.0.0.1`), so other devices on your network cannot reach it |
| **Status** | Shows **Off**, or "Listening on 127.0.0.1:8089, waiting for ATAK" once the server is on, then connected once ATAK joins |
| **Mesh to CoT Converter** | Shows Meshtastic nodes on the ATAK map as contacts. Leave it **off** and you will not see other nodes' positions |
| **TAK Mesh Channel** | Which Meshtastic channel carries outgoing TAK traffic. Choose your **private** channel |
| **Export TAK Data Package** | Creates a `.zip` that ATAK imports to connect to this server |

In the version this guide was checked against, the converter, channel and export options appeared after the server was switched on. If you do not see them, turn the server on first.

**Steps**

1. **Pair the node** in the Meshtastic app (Connect tab) and confirm it shows as connected. Check that the channel matches your other nodes and the role is **TAK**.
2. Open **Settings → TAK Server** and turn on **Enable Local TAK Server**. The status should change to "waiting for ATAK".
3. Turn on **Mesh to CoT Converter**.
4. Under **TAK Mesh Channel**, pick your private channel.
5. Tap **Export TAK Data Package** and save the zip somewhere ATAK can reach (Downloads is fine).
6. In ATAK go to **Settings/Menu → Import Manager → Import → Local SD** and choose the zip. Follow any prompt that appears.
7. Check that the Meshtastic status line changes from "waiting for ATAK" to connected, and that ATAK's server indicator goes green.
8. Keep the **Meshtastic app running** and set its battery use to **Unrestricted** so the link stays up.

**If you already use a real TAK Server:** before importing, note your existing entry under **Settings → Network Preferences → Network Connections**. After importing, confirm your real server entry is still there and still connects. If it was replaced, re-import that server's own enrollment package. ATAK can usually hold more than one server entry, but test this before relying on it.

### 4.3 Option B: the ATAK plugin (matched versions)

The plugin only loads into the ATAK release it was built for, and it expects a Meshtastic app version close to the one it was tested with. Work out the versions first.

**Step 1: Find your ATAK version.** In ATAK open **Settings → About**. Note the version (for example `5.6.0.x`) and the **Plugin API** line (for example `5.6.0.CIV`).

**Step 2: Get the matching plugin APK.** Go to the project's releases page: **https://github.com/meshtastic/ATAK-Plugin/releases**. Each release lists one APK per ATAK version. The version before `-civ-release.apk` is the ATAK version it was built for, for example `…-5.6.0-civ-release.apk`. Pick the one matching your **Plugin API**. Do not pick one for a different version. Each release's notes state what it was tested on (Android version, ATAK version and Meshtastic app version). Note those.

**Step 3: Match the Meshtastic app version.** If the plugin later shows "Failed to bind" (Section 4.7) and the node is connected, the Meshtastic app may be newer than the one the plugin was tested with. In that case install the tested version:

1. **Back up first.** In the current app, save your **channel settings** (channel URL or QR code) and radio configuration. Store the channel key in a password manager, not in a screenshot.
2. Open the Meshtastic app's releases page (**https://github.com/meshtastic/Meshtastic-Android/releases**) and find the tested version.
3. From that release's **Assets**, download **`app-google-release.apk`**. Skip `app-google-release.aab` (an upload bundle that cannot be installed directly), the `fdroid` build (F-Droid store version) and the source code files.
4. **Uninstall the Play Store version first.** The Play Store and GitHub builds are signed differently, so Android will not install one over the other. Uninstalling clears the app's saved messages and settings.
5. Install the APK. Android asks you to allow installs from your browser or file manager. Allow it **only for this install**, then switch it off again.
6. In Google Play, turn **auto-update off** for the Meshtastic app (app page → three-dot menu → untick auto-update), or it will replace your chosen version.
7. Reconnect your node, restore your channel and set the role to **TAK**.

> An older Meshtastic app will not have the built-in TAK Server screen, so this is a choice between the two options, not a way to run both. Try it on a **spare phone** first if you can.

**Step 4: Install and load the plugin.**

1. Install the plugin APK (allow unknown installs for this one install only, as above).
2. Make sure the Meshtastic app is **running and connected** to your node.
3. Open ATAK → **Plugins manager** and load the Meshtastic plugin. Allow any permissions it asks for.
4. Look for the Meshtastic icon at the bottom right of the ATAK map. **Green** means it is bound to the Meshtastic app. **Red** means it is not.

**Step 5: Plugin settings.** Open **Settings → Tool Preferences → Specific Tool Preferences → Meshtastic Preferences** (wording varies by version). Options described in the project's documentation include:

| Setting | What it does |
|---|---|
| **Meshtastic channel index** | Which channel carries ATAK traffic (0 to 7) |
| **Hop limit** | How many radio hops ATAK messages may take |
| **Use Meshtastic GPS** | Use the node's GPS as ATAK's location. If you enable it, also set ATAK's GPS preference to use an external or network GPS only (**Settings → Callsign and Device Preferences → Device Preferences → GPS Preferences**) |
| **Show Meshtastic devices** | Keep this **off** if you use the GPS option, to avoid duplicate markers |
| **Enable relay to server** | Forwards CoT events (except direct messages) to any TAK Server ATAK is connected to |

Check these against the current project documentation, since options change between plugin versions.

> The older community **ATAK Forwarder** plugin is a different project with different version requirements. Do not install both plugins.

### 4.4 Test (both options)

1. Place two devices on the same channel, within radio range.
2. Send a chat message in ATAK. It should reach the other device. **Test replies in both directions.** Some plugin versions have had trouble with messages sent from ATAK.
3. Drop a small marker and confirm it appears on the other device.
4. Confirm the position of a **TAK Tracker** node shows on the ATAK map.
5. Turn on airplane mode (then re-enable Bluetooth only) to prove it works without cell or Wi-Fi.

### 4.5 Using the mesh together with a TAK Server

ATAK can stay connected to your TAK Server **and** use the mesh at the same time. A phone that does both can pass data between the two networks.

- **Mesh to server:** Option B has an **Enable relay to server** setting for this. With Option A, ATAK treats the local server as one more connection. Whether mesh data then reaches your other server should be **tested**, not assumed.
- **Test it:** connect ATAK to both, put a second node nearby, and check whether its position shows for a user connected **only** to the TAK Server. Then test the reverse direction.
- **Use one bridge device.** If two phones are both on the mesh and the TAK Server, the same message can loop back and forth.
- **Mind the bandwidth.** LoRa is slow. A busy server can swamp the mesh. Limit who is connected or use groups.
- **If the link to the server drops,** the mesh keeps working on its own. Mesh users still see each other. Users on the server just stop seeing mesh positions until a bridge reconnects. Test whether messages sent in the meantime are delivered later or lost.
- **Enrollment** with the server is done once on the LAN or VPN. After that the certificate stays on the device.
- Decide on purpose who may see what, and get everyone's consent (Section 6).

### 4.6 ATAK mistakes and fixes

| Mistake | What happens | Correction |
|---|---|---|
| Running the built-in TAK Server and the plugin together | Duplicate markers or conflicts | Use one option at a time |
| Plugin built for a different ATAK version | Plugin missing or crashes | Match the exact ATAK version (check the Plugin API line) |
| Picking the wrong file from the Meshtastic app release | Cannot install | Use `app-google-release.apk`, not the `.aab` or `fdroid` file |
| Installing the GitHub app over the Play Store app | Install fails | Uninstall the Play Store version first, after backing up |
| Auto-update left on | App replaced with a newer version, plugin breaks | Turn off auto-update for the Meshtastic app |
| Two different Meshtastic plugins installed | Conflicts | Use one |
| Meshtastic app killed by battery saver | Link drops | Set to unrestricted, keep running |
| Mesh to CoT Converter left off (Option A) | No nodes on the map | Turn it on |
| Wrong TAK Mesh Channel (Option A) | Traffic goes out on the wrong channel | Select your private channel |
| Node not on the TAK role | Little or no TAK data | Set the role and check the TAK module |
| Different channels or modem presets | No link between nodes | Match them |
| Bluetooth pairing handled outside the app | Plugin shows disconnected | Reconnect in the Meshtastic app, restart the plugin |
| Importing the package over your real server entry | Real server connection broken | Check Network Connections after importing; re-import your server's own package if needed |
| Two bridge phones on the mesh and server | Messages loop | Use one bridge |
| Skipping the range test | Unknown real-world coverage | Test at your actual distances and terrain |

### 4.7 Option B: if the icon stays red ("Failed to bind to Meshtastic IMeshService")

This message means the plugin could not connect to the service inside the Meshtastic app. Try these in order:

1. Open the Meshtastic app and check the node shows as **connected**.
2. Turn off the built-in **Local TAK Server** and **Mesh to CoT Converter**.
3. Set the Meshtastic app's battery usage to **Unrestricted** and do not swipe it out of recents.
4. Force-stop ATAK and the Meshtastic app. Open **Meshtastic first, ATAK second**, then reload the plugin and watch the icon. The plugin retries on its own for a while.
5. Check permissions (Bluetooth or nearby devices, notifications) for both apps.
6. Compare your Meshtastic app version with the version in the plugin release notes. If yours is much newer, install the tested version as described in Step 3 above. **This is what fixed it** for this guide's author: the current app gave "Failed to bind", and a 2.7 version of the app worked with the 5.6.0 plugin build.

---

## 5. iTAK (iPhone / iPad)

iOS does not allow plugins in the way Android does. The Meshtastic iOS app solves this by running a **local TAK Server inside the app**. iTAK connects to that local server on the same device, and the app passes traffic to the mesh.

### 5.1 What you need

| Item | Notes |
|---|---|
| iPhone/iPad with **iTAK** working | |
| **Meshtastic iOS app**, recent version | This feature is newer. Update the app and check its release notes |
| A Meshtastic node paired by Bluetooth | |
| Node role | TAK or TAK Tracker |

### 5.2 Steps

1. **Pair the node** in the Meshtastic app and confirm the channel matches your other nodes.
2. In the Meshtastic app open **Settings** and scroll to **TAK Server** (wording varies by version).
3. Set your **TAK identity**: team and role.
4. Turn on **Enable TAK Server**. Check the status indicator. It may warn if the primary channel is unsuitable for TAK use.
5. **Export the TAK data package** (zip) from the same screen.
6. Open the zip with **iTAK** (Files → Share → iTAK, or Open in iTAK) and import it. This tells iTAK how to find and trust the local server, with no address or certificate typing.
7. Confirm iTAK shows the connection as **connected**.
8. Keep the **Meshtastic app running** and allow Bluetooth and background activity so the link stays up.

The local server is meant to be reachable only from the same device, so it is not exposed to your network.

### 5.3 Things that behave differently

- **iTAK is connected to the local server.** If you also use a real TAK Server, check how your iTAK version handles more than one server at a time before relying on both.
- **Routes:** iTAK ignores route data that arrives over its streaming connection. The Meshtastic app saves received routes as a data package you can import by hand (look under **Files → Meshtastic → TAK Routes**, then share the zip to iTAK and import it as a mission package).
- Features and wording change quickly. Check the current Meshtastic iOS documentation.

### 5.4 Test

1. Send a chat from iTAK. It should arrive on another node's app.
2. Send one back and confirm it arrives in iTAK.
3. Confirm positions of other nodes show on the iTAK map.
4. Test with no cell or Wi-Fi (airplane mode, then Bluetooth on).

### 5.5 iTAK mistakes and fixes

| Mistake | What happens | Correction |
|---|---|---|
| Local TAK Server not enabled | iTAK cannot connect | Enable it in the Meshtastic app |
| Importing a zip unzipped by Files | iTAK doesn't import it | Share the original zip to iTAK |
| Meshtastic app closed or suspended | Link drops | Keep it running; allow background activity |
| Expecting routes to appear automatically | Nothing shows | Import the saved route package |
| Old Meshtastic app version | Feature missing | Update the app |

---

## 6. Security and privacy (all clients)

- **Use a private channel with your own random key.** Never use the default public channel for TAK traffic.
- **Everyone with the channel key can read positions and chat on it.** Treat the key like a password. Do not post QR codes or screenshots of channel settings.
- **LoRa signals can be received by any node in range with the key**, and the fact that radio is transmitting can be noticed by anyone nearby.
- **Bridges connect networks.** A device linked to both your TAK Server and the mesh can pass data between them. Decide on purpose what crosses.
- **Share only what you must.** Position precision, callsigns and team names can all identify people. Use neutral names.
- Keep firmware, apps and bridge software updated, and only install them from official sources.
- Get the **informed consent** of everyone whose position is shared.

## 7. Final checklist (per device)

- [ ] Node has current firmware, correct region and role
- [ ] Private channel with a fresh key loaded on every node
- [ ] Bridge installed for this client (WinTAK gateway / ATAK built-in TAK Server or plugin / iOS local TAK server)
- [ ] Bridge shows connected
- [ ] Callsign, team and role set
- [ ] Position shows on a second device's map
- [ ] Chat works both ways
- [ ] Tested with **no** cell, Wi-Fi or internet
- [ ] Tested at your real distances and terrain
- [ ] Reboot test: restart everything and confirm it reconnects
- [ ] Spare power for nodes and phones
- [ ] You do not rely on this as your only communication or navigation method

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| Nodes don't see each other | Channel name, key or modem preset differ; region wrong; out of range |
| Chat arrives one way only | One side's bridge or role misconfigured; check both ends |
| No position on the map | GPS off, no fix indoors, or role not TAK/TAK Tracker |
| Bridge says disconnected | Bluetooth/USB link lost; reconnect, then restart the bridge |
| Works near, fails far | Terrain, antenna, preset. Test and adjust |
| Messages are slow or missing | LoRa airtime is limited; too many nodes or traffic. Send less, shorten messages |
| WinTAK gateway silent | Multicast off, firewall blocking, COM port busy, wrong node address |
| Plugin missing in ATAK | Version mismatch with ATAK |
| ATAK plugin icon is red, "Failed to bind to Meshtastic IMeshService" | Meshtastic app not running or not connected, or app version differs from the one the plugin was tested with. See Section 4.7 |
| ATAK shows no nodes (built-in TAK Server) | Mesh to CoT Converter is off, or the data package was not imported |
| iTAK can't connect | Local TAK server off, wrong or outdated import, Meshtastic app not running |

---

## Credits and links

- TAK Product Center (ATAK, WinTAK, iTAK, TAK Server): https://tak.gov
- Meshtastic project and documentation: https://meshtastic.org
- Meshtastic ATAK Plugin releases: https://github.com/meshtastic/ATAK-Plugin/releases
- Meshtastic Android app releases: https://github.com/meshtastic/Meshtastic-Android/releases
- TAK Meshtastic Gateway (community project): search the name on https://pypi.org

*Meshtastic firmware, apps and bridges change often. Menu names and options may differ from this guide. Verify against current documentation and test with two nodes before relying on any step.*

---

## Legal notice, license and disclaimer

**This is general boilerplate, not legal advice.** If you need certainty about your obligations, ask a qualified lawyer.

### Use at your own risk; no warranty

This guide, and any scripts, configuration files or other code distributed with it (together, the "Materials"), are provided **"as is" and "as available," without warranty of any kind**, express or implied. That includes, without limitation, any warranty of merchantability, fitness for a particular purpose, accuracy, completeness, security, reliability or non-infringement.

To the fullest extent permitted by law, the authors and contributors are **not liable** for any claim, damages or other liability, whether in contract, tort or otherwise, arising from or connected with the Materials or their use. This includes data loss, corrupted or wiped storage, hardware damage, downtime, security incidents, exposure of data or locations, financial loss, injury, or any other harm. You follow this guide entirely at your own risk.

### Accuracy and AI assistance

Parts of this guide were drafted with the help of an AI assistant and may contain errors, omissions or outdated steps. Software, firmware, plugins, download locations and menu names change between versions. Always check against the official documentation for Meshtastic, TAK clients and any bridge software you use, and test on non-critical equipment first.

### Commands and installs can be destructive

Several steps install software, run third-party programs, change device settings or flash firmware. A mistake can erase data, brick a radio or lock you out of a device. Back up anything you care about first, and read every command and script before you run it.

### Not affiliated; trademarks

This is an unofficial, independent community guide. It is **not affiliated with, endorsed by or sponsored by** the TAK Product Center, the Meshtastic project, any government agency, Apple, Google, Microsoft, or any other company or project mentioned. TAK, ATAK, WinTAK, iTAK, TAK Server, Meshtastic, LoRa, Android, iOS, iPhone, iPad, Windows, Bluetooth and all other names, logos and trademarks belong to their respective owners and are used here only to identify the products.

### Third-party software and content

- **TAK Server and TAK clients** are not included. Obtain them from the official source (tak.gov, Google Play or the App Store) and follow the terms you accept there. Do not redistribute them.
- **Meshtastic software and firmware, plugins, gateways and other third-party tools** have their own licenses, maintainers and risks. You are responsible for reviewing and complying with them. This guide does not audit or guarantee them.
- Links point to external sites that the authors do not control.

### Security

This guide covers **basic setup**, not a hardened deployment. It makes no promise that any resulting setup is secure or private. Radio transmissions can be received by others, and anyone with your channel key can read what is sent on it. Protect keys, certificates, passwords and backups. You are responsible for securing your systems and the data on them.

### Radio regulations, privacy, consent and the law

Operating a radio transmitter is regulated. You are responsible for complying with all laws and rules that apply to you, including radio and telecommunications rules (frequency band, region, power and antenna limits, and any licensing), privacy and data-protection law, consent from the people whose location and data are shared, employer or organisational policies, and export-control and other legal requirements for the software and hardware you use. Do not use these Materials for any unlawful purpose, including to track anyone without their knowledge and consent.

### Not for safety-critical use

Do not rely on a mesh radio, a phone, a computer or any software described here as your **only** means of navigation, communication, tracking or emergency response. Range is limited, messages can be delayed or lost, equipment fails, batteries die and data can be wrong or stale. Carry independent backups (such as a proper emergency communication device, paper maps and a compass), test your setup in advance, and use sound judgment, especially outdoors or in any situation where a failure could put someone at risk.

### License

Unless stated otherwise, and subject to the disclaimers above:

- **The text of this guide** is licensed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license.
- **Any code, scripts or configuration files** distributed with this guide are licensed under the **MIT License**.

Copyright (c) 2026 4head-tac-tech. Third-party software and content keep their own licenses. By contributing, you agree that your contribution is licensed on the same terms. The warranty disclaimer and limitation of liability above apply to everyone who uses, copies, modifies or redistributes the Materials.

### Reporting problems

Found a mistake, a security issue or an outdated step? Please open an issue or a pull request in the project repository: https://github.com/4head-tac-tech/TAK-Field-Kit.
