# Meshtastic Node to WinTAK over Wi-Fi: Windows Gateway Setup (Hands-On Guide)

A step-by-step walkthrough for linking a **Meshtastic** node on your home network to **WinTAK** on a Windows PC, using the community **TAK Meshtastic Gateway**. It covers everything between "I have a node and a PC" and "the gateway is running", including the install errors you are likely to hit and how they were fixed.

This is a **companion** to the *Meshtastic with WinTAK, ATAK and iTAK* guide. That guide gives the overview for all three clients. This one goes deep on **WinTAK with a node reached over Wi-Fi/LAN**.

> **Use at your own risk. No warranty.** This guide is an unofficial, community-written resource provided "as is." It is not affiliated with or endorsed by the TAK Product Center or the Meshtastic project. Read the full **[Legal notice, license and disclaimer](#legal-notice-license-and-disclaimer)** at the end before you follow it. A radio mesh broadcasts your **live location** to every node in range that shares your channel, and this setup puts your WinTAK position on that mesh. Know who is on your channel.

---

## Placeholders used in this guide

| Placeholder | Meaning |
|---|---|
| `<NODE_IP>` | The Meshtastic node's address on your LAN (give it a DHCP reservation) |
| `<PC_IP>` | The Windows PC's address on the same LAN |
| `<CHANNEL_NAME>` | Your private Meshtastic channel name |
| `<PSK>` | The channel's encryption key. Never publish it |
| `<PRECISION>` | The position precision you choose for the channel (see Step 6) |
| `<CALLSIGN>` | The name you appear as on the TAK map |
| `%USERPROFILE%` | Your Windows user folder. Windows fills this in for you |

Never publish your real values, and never post screenshots of the node's channel settings. The key is printed in plain text by several commands used below.

---

## Versions used (a starting point)

These are the versions this guide was put together with. Use them as a **place to start**, not a guarantee. Newer versions may change commands, menus or behaviour.

| Item | Version used | Notes |
|---|---|---|
| **Windows** | Current 64-bit Windows | Command Prompt (`cmd`) is used throughout |
| **Python for the gateway** | **3.12.10** | The gateway project asks for 3.12 on Windows. **3.15 failed to install it** (see Step 3) |
| **TAK Meshtastic Gateway** | **1.1.0** | Installed from PyPI with `pip` |
| **Meshtastic Python library / CLI** | **2.7.11** | Installed automatically as a dependency of the gateway |
| **takproto** | **3.0.1** | Pulled in automatically. See the note in Step 3 |
| **Node hardware** | ESP32-S3 board with GPS (LILYGO T-Beam S3 Core) | Any Wi-Fi capable Meshtastic node should behave the same |
| **Node firmware** | **2.7.x** | The older V1 TAK format: position and chat only |
| **WinTAK** | WinTAK-CIV | Menu names below come from this build and may differ in yours |

### What has and has not been confirmed

**Confirmed on the author's setup:**
- The node connected over Wi-Fi from the PC with the Meshtastic CLI.
- The role was set to **TAK** and read back correctly.
- A private channel was made the primary channel.
- The gateway installed, connected to the node, and logged a "Sending position" / "Sent ATAK packet" pair every 30 seconds.
- A second node on the same channel appeared as a contact on the WinTAK map, so at least position data reached WinTAK from the mesh.
- **A chat message sent from WinTAK was delivered to an ATAK device over the mesh.** The WinTAK-to-mesh chat path works.
- **A test message sent from a phone on the mesh was received in WinTAK.** The mesh-to-WinTAK chat path works too, so chat has been confirmed in both directions.

**Not confirmed:**
- **Why the first WinTAK chat attempts stayed on a "sending" spinner, and what changed before the message was delivered.** The causes in Step 9 are theories to test, not findings.
- Which channel the gateway transmits on. Treat it as the **primary channel** and confirm for your version.
- That the gateway's WinTAK settings need no changes. The author changed none.

---

## 1. How it fits together

```
WinTAK  <-- multicast on the PC -->  TAK Meshtastic Gateway (on the PC)
                                              |
                                      TCP over your LAN
                                              |
                                      Meshtastic node  <-- LoRa radio -->  other nodes
```

| Piece | What it does |
|---|---|
| **WinTAK** | The map, chat and contacts. Broadcasts and listens on standard TAK multicast addresses |
| **TAK Meshtastic Gateway** | A small program on the PC. Listens for WinTAK's multicast data and passes position and chat to the node, and passes mesh traffic back |
| **Meshtastic node** | The radio. Reached over Wi-Fi at `<NODE_IP>`, so it can sit near a window or outside while the PC stays inside |

**Limits:**
- The gateway carries **position (PLI) and chat only**. Markers, shapes, routes, images and data packages do not cross the mesh. On firmware older than 2.8.0 this is a firmware limit as well.
- The gateway only works **while its window is open and the PC is awake**. If the PC sleeps, the node keeps relaying the mesh on its own, but WinTAK stops receiving anything until the bridge runs again.
- A node accepts **one connected client at a time** (usually). Close the Meshtastic phone app and any other client before the PC connects.

---

## 2. Before you start

- [ ] WinTAK is installed and working on its own.
- [ ] The node has current firmware, the **correct radio region** set, and is on your Wi-Fi.
- [ ] You know `<NODE_IP>` and gave it a **DHCP reservation** in your router so it never changes.
- [ ] The node has enough battery or is on USB power. A node on a low battery (around 3.5 V on a LiPo) can drop out or reboot.
- [ ] You have **at least one other node** to test with.
- [ ] You decided who gets the channel key (Step 6).

---

## 3. Step 1: confirm the PC can reach the node

1. Disconnect or close the Meshtastic phone app (tap **Disconnect**, then close it fully).
2. In Command Prompt, run:
   ```
   ping <NODE_IP>
   ```
   You want replies with **0% loss**. If it times out, the PC and node are on different networks (guest Wi-Fi, VPN, second router) and nothing below will work until that is fixed.

> **Why not the browser?** The Meshtastic web client at `client.meshtastic.org` is served over HTTPS, and browsers commonly block it from talking to a plain-HTTP device on your LAN. That is why this guide uses the command-line tool instead. It was the likely reason the web client would not connect in testing. Firefox also does not support the web client's USB and Bluetooth options.

---

## 4. Step 2: install Python 3.12

You need **Python 3.12** for the gateway. The newest Python (3.15 at the time of writing) failed, because one of the gateway's helper packages had no ready-made build for it.

1. Download the installer from **python.org/downloads**. The current installer is the **Python install manager**. Windows may show a generic "unknown publisher" warning for any downloaded installer. Check that the publisher is *Python Software Foundation* and the download came from python.org.
2. Run it. It asks a few questions:
   - **Add the Python commands folder to PATH:** answer **yes**.
   - **Install a Python runtime:** answer **yes**.
   - Optional extras and online help: answer **no**.
   - If it warns that an **older `py` launcher is already installed**, answer **no** to removing it. You do not need to uninstall anything.
3. **Close every Command Prompt window and open a new one.** PATH changes only apply to new windows.
4. Install Python 3.12 alongside whatever version was installed first:
   ```
   pymanager install 3.12
   ```
   Use `pymanager`, not `py install`. On a PC that already had the old launcher, `py install` is read as a file name and fails.
5. Check it:
   ```
   python3.12 --version
   ```
   It should print `Python 3.12.x`.

---

## 5. Step 3: create a virtual environment and install the gateway

A virtual environment ("venv") keeps this project's packages separate from the rest of the PC.

```
cd %USERPROFILE%
python3.12 -m venv tak_gw
dir tak_gw\Scripts\pip.exe
```

If `dir` lists `pip.exe`, install the gateway:

```
tak_gw\Scripts\pip install tak-meshtastic-gateway
```

> **Typing pitfall:** `C:Users\name\tak_gw` (no backslash after `C:`) is a *relative* path. Windows creates the venv in the wrong place and later commands cannot find it. Use `cd %USERPROFILE%` and the short name `tak_gw` as above.

### If the install fails: "Microsoft Visual C++ 14.0 or greater is required"

This was the error for the helper package `unishox2-py3`, which pip had to compile because no prebuilt version matched.

1. Try upgrading pip first, then retry. This sometimes finds a prebuilt version:
   ```
   tak_gw\Scripts\python.exe -m pip install --upgrade pip
   tak_gw\Scripts\pip install tak-meshtastic-gateway
   ```
2. If it still fails, install Microsoft's free compiler from **https://visualstudio.microsoft.com/visual-cpp-build-tools/**. In the installer, **tick "Desktop development with C++"** and click Install. The base tools alone (about 137 MB) are not enough. The full workload is a few GB and takes several minutes.
3. Close all Command Prompt windows, open a new one, and retry.

### If it then fails with error C1083 (cannot open a file)

This is a **path-length problem**. pip builds in a very long temporary folder, and Windows tools can choke on paths over about 260 characters. Build from a short folder instead. Run these in the **same** window:

```
mkdir C:\t
set TMP=C:\t
set TEMP=C:\t
tak_gw\Scripts\pip install --no-cache-dir tak-meshtastic-gateway
```

The two `set` commands last only for that window. They change nothing permanently. If you open a new window, run them again before pip.

A successful install ends with a line starting **"Successfully installed"** that includes `tak-meshtastic-gateway`.

> **Note on `takproto`.** The gateway's project page describes a bug in the `takproto` library that can cause exceptions when parsing XML CoT data, and gives an extra `pip install git+https://github.com/...` line as a workaround that also needs Git. On the author's setup pip installed `takproto 3.0.1` by itself and the gateway ran without that extra step. If you later see XML parsing exceptions in the gateway window, check the project's page for the current workaround.

> **Read before you run.** The gateway is a third-party project installed from the internet. This guide does not audit it. The install steps change, so trust the project's own page (search **TAK Meshtastic Gateway** on pypi.org) over this guide.

### The Meshtastic command-line tool comes with it

The gateway installs the Meshtastic Python library as a dependency, so the CLI is already in the same venv. You can run it as:

```
tak_gw\Scripts\python.exe -m meshtastic --host <NODE_IP> --info
```

> The author originally ran the CLI from a separate Python 3.13 install (`pip install meshtastic`). Running it from the gateway's venv is the simpler path and uses the same package, but was not separately tested. If it does not work, installing the CLI separately works.

---

## 6. Step 4: make short launchers

Typing the long commands every time invites typos. These two small files shorten them.

```
echo @%USERPROFILE%\tak_gw\Scripts\python.exe -m meshtastic --host <NODE_IP> %* > %USERPROFILE%\mesh.bat
echo @%USERPROFILE%\tak_gw\Scripts\tak-meshtastic-gateway --mesh-ip <NODE_IP> %* > %USERPROFILE%\gw.bat
```

(You can create the same two files in Notepad instead. Use "Save as type: All Files" so they do not become `.txt`.) With your Command Prompt open in your user folder:

| You type | What it runs |
|---|---|
| `mesh --info` | The Meshtastic CLI against your node |
| `mesh --get device.role` | Reads one setting |
| `gw` | Starts the gateway |
| `gw -d` | Starts the gateway with debug output |

`mesh` and `gw` are found only from your user folder unless you add it to PATH. If the gateway's file is not found, run `dir tak_gw\Scripts | findstr /i gateway` and use the exact name shown.

Test it:
```
mesh --info | findstr /i "firmware"
```

> **Careful with `mesh --info`.** It prints the whole configuration, including your **channel keys** and every node your radio has heard. Do not screenshot or paste it. Use `--get <setting>` for single values.

---

## 7. Step 5: set the node role to TAK

The gateway's documentation says the node should be in the **TAK** role. Back up the configuration first. The backup file contains your keys, so keep it private:

```
mesh --export-config > %USERPROFILE%\mesh-backup.yaml
```

Then set the role (phone app closed):

```
mesh --set device.role TAK
```

You should see "Set device.role to TAK" and "Writing modified preferences to device". The node **reboots and takes about 30 seconds to come back**. Wait, then check:

```
mesh --get device.role
```

The CLI prints a number: **`device.role: 7` means TAK.**

> **Typing pitfall:** `--get device role` (with a space) gives a usage error. It must be `device.role` with a dot.

**What the TAK role does:** it makes the node optimised for TAK use and reduces routine broadcasts. It is a good fit for a node paired with a TAK client. For a standalone node that only reports its own position with no TAK device attached, the **TAK Tracker** role is the one meant for that.

---

## 8. Step 6: make a private channel the primary channel

**Why this matters:** the gateway forwards your WinTAK position and chat to the mesh, and as far as testing shows it does so on the node's **primary channel (index 0)**. A new node's primary channel is usually the **default public channel** with a well-known key, so anyone with a Meshtastic node in range could read your position. Put a private channel in the primary slot first.

Primary is always channel index 0, so you do not "promote" a channel. You **write your private channel's name and key onto index 0**.

**Fresh private channel** (phone app closed):

```
mesh --ch-index 0 --ch-set name <CHANNEL_NAME> --ch-set psk random --ch-set module_settings.position_precision <PRECISION>
```

`psk random` generates a new random key. (Check the current Meshtastic CLI documentation if your version rejects it.) To reuse an **existing** key instead, use `--ch-set psk base64:<PSK>`.

If you previously had a second copy of the channel in slot 1, remove it:

```
mesh --ch-del --ch-index 1
```

Wait about 30 seconds for the node, then check:

```
mesh --info | findstr "Index"
```

You should see **Index 0: PRIMARY** with your channel name and a `psk` that is not the default. Do not share the output.

### Choosing position precision

| Value | Effect |
|---|---|
| **32** | Full precision. Your exact position goes to everyone with the key |
| Lower numbers | Blurs the position to a larger grid. The default public channel uses a coarse value (13), which can make a node appear **about a mile or more off** |

Pick this on purpose. Full precision is reasonable for a small private group that has agreed to it. If a node appears about a mile off on the map, check this value for that node's channel first.

### What changes when you leave the default channel

- Your node most likely **stops hearing and being heard by the public mesh**. The channel name also sets the radio's frequency slot, so a new name normally means a different frequency. This is expected, and it is the point. Check with `mesh --nodes`: the list should shrink to your own nodes.
- Other nodes that should talk to you need the **same channel name, key and modem preset**. Share the channel link privately. It contains the key:
  ```
  mesh --info | findstr /i "URL"
  ```
- Packets from other nodes on frequencies you cannot hear are encrypted to anyone without your key, but **who** is transmitting and when is still visible to anyone who can receive the signal.
- If you ever expose the key (a screenshot, a post, a shared file), generate a new one with `--ch-set psk random` and re-share the channel link with your group.

---

## 9. Step 7: check WinTAK's network settings

The gateway listens for WinTAK's **multicast** traffic, so WinTAK must be sending and receiving it. In WinTAK open **Settings (Preferences) → Network Preferences**. In the version used, these entries appeared under **Input/Output Management**.

**Manage Inputs.** All of these were present and ticked by default:

| Name | Address |
|---|---|
| Default / Default TCP | `udp://0:4242` / `tcp://0:4242` |
| SA Multicast | `udp://239.2.3.1:6969` |
| GeoChat Messages | `udp://224.10.10.1:17012` |
| GeoChat Announce | `udp://224.10.10.1:18740` |

**Manage Outputs.** Present and ticked by default:

| Name | Address |
|---|---|
| Chat output | `udp://224.10.10.1:17012` |
| SA Multicast | `udp://239.2.3.1:6969` |

The author changed **none** of these, and the gateway still connected and sent position. If an entry is missing or unticked on your PC, restore the default and tick it.

**Manage Network Interfaces.** Multicast only leaves through the adapters WinTAK is allowed to use. If the PC has several (Wi-Fi, Ethernet, VPN, virtual adapters), make sure the adapter on `<PC_IP>` is enabled.

**Windows Firewall.** Allow WinTAK and Python on **private** networks. The Python program to allow is the **real** one the venv points to (check the gateway window's error traceback if it shows a path such as `...\Python\pythoncore-3.12-64\`).

**Callsign and position.**
- Set your **callsign, team and role** in WinTAK.
- A PC with no GPS shows **NO GPS** in WinTAK's bottom-left information box. The gateway forwards WinTAK's own marker, **not the node's GPS**, so with no GPS fix there is nothing real to send. Set a **manual position** for your own marker (look for a GPS or location option in WinTAK's settings or on that box; the exact menu varies by version).
- **Do not rely on the node's GPS for WinTAK's own marker.** No setting was found that feeds the node's GPS into WinTAK. The node still broadcasts its own position on the mesh in the TAK role, so you may see two nearby dots for the same place.

---

## 10. Step 8: run the gateway

1. Close the Meshtastic phone app (fully, not only Disconnect).
2. Start **WinTAK**.
3. Start the gateway:
   ```
   gw
   ```
   or the full command:
   ```
   %USERPROFILE%\tak_gw\Scripts\tak-meshtastic-gateway --mesh-ip <NODE_IP>
   ```

**Useful options** (run `tak-meshtastic-gateway --help` for your version):

| Option | What it does |
|---|---|
| `-d` | Debug output. Use this when troubleshooting |
| `-l` | Write a log file |
| `--tx-interval` | How often position is sent (default 30 seconds) |
| `-p` / `--dm-socket-port` | TCP port the gateway listens on for direct messages (default 4243) |

**What a healthy start looks like:**
1. `Your IP address is <PC_IP>`
2. `Connected to the Meshtastic Device`
3. `Starting TAK-Meshtastic-Gateway 1.1.0`
4. Every 30 seconds: `Sending position to Meshtastic` followed by `Sent ATAK packet to Meshtastic`

**Leave the window open.** Closing it stops the bridge. It has to be restarted after the node reboots (for example after a role or channel change).

### An occasional red error that heals itself

You may see:
```
Unexpected OSError, terminating meshtastic reader... [WinError 10054]
An existing connection was forcibly closed by the remote host
```
followed by `Lost connection to the Meshtastic device, attempting to reconnect...` and then `Connected to the Meshtastic Device` about a second later. A Python traceback ending in `ConnectionResetError: [WinError 10054]` can appear in the same way. If the gateway reconnects and keeps sending, this is the node dropping the TCP link and is not a fault. Causes to rule out if it happens often:

- **The phone app reconnected to the node.** The node accepts one client at a time. Fully close the app.
- **Low battery or weak Wi-Fi at the node.** Power it from USB and check signal strength.

---

## 11. Step 9: test

1. Put a second Meshtastic node, **on the same channel name, key and modem preset**, within radio range. Pair it to a phone running the Meshtastic app.
2. **Position.** Look at WinTAK's map. The second node should appear as a contact after its next position broadcast. It shows under the node's **long name** (the default is "Meshtastic" plus four characters). To rename it, use the app (Settings → Radio configuration → User) or the CLI against that node:
   ```
   meshtastic --host <OTHER_NODE_IP> --set-owner "<NAME>" --set-owner-short <SHORT>
   ```
   The new name appears after the node's next position update.
3. **Chat from the mesh to WinTAK.** Send a text in the Meshtastic app. Look for it in WinTAK's chat.
4. **Chat from WinTAK to the mesh.** Open Contacts and test **All Chat Rooms** first, then a direct chat to the mesh contact. Look for the text in the Meshtastic app.
5. **Watch the gateway window** while you send. New lines mean the message reached it.

### Known issue: WinTAK chat to a mesh contact stuck on a spinner

On the author's setup a direct message to the mesh contact kept a small spinner next to its timestamp and was not delivered.

**Update:** a WinTAK chat message was later delivered to an ATAK device over the mesh, so the path does work. What changed between the stalled attempts and the successful one was not recorded. The list below stays as things to check if your own messages stall. These are **theories to test**, in this order. None is confirmed:

1. **Test the broadcast path first.** Send to **All Chat Rooms**. Broadcast chat goes over the GeoChat multicast address (`224.10.10.1:17012`) that the gateway is documented to listen to. If this works but a direct chat does not, the problem is specific to direct messages.
2. **Direct messages probably use a TCP port on the gateway** (the project documents a "DM socket" on port 4243). With the gateway running, open a second Command Prompt:
   ```
   netstat -ano | findstr 4243
   ```
   - **No output:** the gateway is not listening. Check its startup lines.
   - **A line with `LISTENING`:** the gateway is listening, so **Windows Firewall** may be blocking the inbound connection. Allow the real `python.exe` on **private** networks (Windows Security → Firewall & network protection → Allow an app through firewall).
3. **Start the gateway with `-d`** and send again, to see whether anything reaches it at all.
4. **Network Interfaces.** Check that WinTAK uses the right adapter (Step 7).
5. WinTAK has a **Gateway System Interoperability** checkbox under Network Tweaks whose description says that, on certain gateway systems, directed messages need relay information. It is **off by default**. Whether it applies here is unknown. Try it only if everything above checks out and direct chat still fails.

---

## 12. Common mistakes and corrections

| Mistake | What happens | Correction |
|---|---|---|
| Using Python 3.15 (or the newest) for the gateway | A helper package has no prebuilt version and pip tries to compile it | Use **Python 3.12** |
| `py install 3.12` on a PC with the old launcher | Treated as a file name and fails | Use `pymanager install 3.12` |
| Not opening a **new** Command Prompt after installing | `python` not found | Close and reopen so PATH is read again |
| `C:Users\...` with no backslash | Venv created in the wrong place | `cd %USERPROFILE%`, then `python3.12 -m venv tak_gw` |
| Base Build Tools installed without the C++ workload | Compile error persists | Tick **Desktop development with C++** |
| Compile error C1083 after installing the tools | Path too long | Set `TMP` and `TEMP` to `C:\t` for that window |
| `--get device role` (space) | Usage error | `--get device.role` (dot) |
| Phone app still connected to the node | PC cannot connect, or drops | Close the app fully |
| Running `mesh --info` and screenshotting it | Channel key exposed | Use `--get`, never post the output |
| Leaving the default public channel as primary | Your position goes out on a public-key channel | Put a private channel on index 0 (Step 6) |
| Closing the gateway window | Bridge stops | Leave it running |
| Not restarting the gateway after the node reboots | Gateway talks to a dead session | Restart it |
| PC with no GPS and no manual position | Wrong or no position shared | Set a manual position in WinTAK |
| Expecting markers, files or maps over the mesh | Nothing arrives | The gateway carries position and chat only |
| Node address changed | Gateway and CLI cannot find it | Set a DHCP reservation |
| PC goes to sleep | Bridge stops, WinTAK gets nothing | Disable sleep while plugged in, or move the bridge to an always-on device |
| Node with low battery | Dropouts, reboots, weak transmit | Power it from USB |

## 13. Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| `ping <NODE_IP>` times out | Different network, wrong address, or node off Wi-Fi |
| CLI or gateway cannot connect | Phone app holding the node; node rebooting (wait 30 s); wrong address |
| Gateway connects, then drops every few minutes | Battery, Wi-Fi signal, or another client; see Step 8 |
| Gateway log never shows "Sending position" | WinTAK not running or not sending multicast; check Step 7 |
| No mesh contacts on the WinTAK map | Other node on a different channel, key or modem preset; out of range; no GPS fix; role not TAK or TAK Tracker |
| A contact appears about a mile off | Low position precision on its channel, or no GPS fix. See Step 6 |
| Contact shows as "Meshtastic xxxx" | Default long name. Rename it (Step 9) |
| WinTAK chat spins and is not delivered | See "Known issue" in Step 9 |
| XML parsing exceptions in the gateway window | The `takproto` bug. Check the gateway project's page |
| Public-mesh nodes vanished after Step 6 | Expected. A new channel name moves you to a different frequency slot |

## 14. Security and privacy

- **Use a private channel with your own random key.** Never use the default public channel for TAK traffic.
- **Everyone with the key can read your position and chat on that channel.** Treat the key like a password. Do not post the channel link, QR code, `--info` output or the exported configuration file.
- The gateway **forwards your WinTAK position to the mesh**. Choose the position precision on purpose, and get the **consent** of everyone whose position is shared.
- Radio transmissions can be received by anyone in range, and the fact that a transmitter is active can be noticed even without the key.
- Install software only from official sources, and keep firmware, WinTAK and the gateway updated.
- This guide covers a **basic home setup**, not a hardened deployment.

## 15. Final checklist

- [ ] `ping <NODE_IP>` shows 0% loss; node has a DHCP reservation
- [ ] Python 3.12 installed; venv created in the right place
- [ ] `tak-meshtastic-gateway` installed ("Successfully installed")
- [ ] `mesh` and `gw` launchers work
- [ ] Configuration backed up (and stored privately)
- [ ] Node role is TAK (`device.role: 7`)
- [ ] Private channel is **Index 0, PRIMARY**, with a non-default key and a precision you chose
- [ ] Channel link shared privately with everyone who needs it
- [ ] WinTAK Inputs and Outputs entries present and ticked; correct network adapter enabled
- [ ] Firewall allows WinTAK and the real `python.exe` on private networks
- [ ] WinTAK callsign set; manual position set if the PC has no GPS
- [ ] Gateway shows "Connected" and repeated "Sent ATAK packet"
- [ ] Second node appears on the WinTAK map
- [ ] Chat tested both ways, and the result written down
- [ ] Restart test: reboot the node and gateway and confirm it all reconnects
- [ ] You do not rely on this as your only communication or navigation method

## 16. What comes next

- **Run the gateway on an always-on device** (for example a Raspberry Pi) so the PC can sleep without breaking the bridge. The gateway's project notes that a node on the LAN can be reached without USB, which fits this. This guide did not test it.
- **Firmware 2.8.0 or newer** moves the TAK data to a newer format that adds shapes, markers and routes. Check the current Meshtastic documentation before upgrading, and confirm that the gateway supports it.
- **ATAK and iTAK** each use a different bridge. See the *Meshtastic with WinTAK, ATAK and iTAK* guide.

---

## Credits and links

- TAK Product Center (ATAK, WinTAK, iTAK, TAK Server): https://tak.gov
- Meshtastic project and documentation: https://meshtastic.org
- Meshtastic TAK integration documentation: https://meshtastic.org/docs/software/android/user/tak/
- TAK Meshtastic Gateway (community project, OpenTAKServer): https://pypi.org/project/TAK-Meshtastic-Gateway
- Python downloads: https://www.python.org/downloads/
- Microsoft C++ Build Tools: https://visualstudio.microsoft.com/visual-cpp-build-tools/

*Meshtastic firmware, apps and bridges change often. Menu names, commands and options may differ from this guide. Verify against current documentation and test with two nodes before relying on any step.*

---

## Legal notice, license and disclaimer

**This is general boilerplate, not legal advice.** If you need certainty about your obligations, ask a qualified lawyer.

### Use at your own risk; no warranty

This guide, and any scripts, configuration files or other code distributed with it (together, the "Materials"), are provided **"as is" and "as available," without warranty of any kind**, express or implied. That includes, without limitation, any warranty of merchantability, fitness for a particular purpose, accuracy, completeness, security, reliability or non-infringement.

To the fullest extent permitted by law, the authors and contributors are **not liable** for any claim, damages or other liability, whether in contract, tort or otherwise, arising from or connected with the Materials or their use. This includes data loss, corrupted or wiped storage, hardware damage, downtime, security incidents, exposure of data or locations, financial loss, injury, or any other harm. You follow this guide entirely at your own risk.

### Accuracy and AI assistance

Parts of this guide were drafted with the help of an AI assistant and may contain errors, omissions or outdated steps. Software, firmware, packages, download locations and menu names change between versions. Always check against the official documentation for Meshtastic, WinTAK, Python and any bridge software you use, and test on non-critical equipment first.

### Commands and installs can be destructive

Several steps install software, run third-party programs, change device settings or alter a radio's configuration. A mistake can erase data, lock you out of a device, or expose your location. Back up anything you care about first, and read every command before you run it.

### Not affiliated; trademarks

This is an unofficial, independent community guide. It is **not affiliated with, endorsed by or sponsored by** the TAK Product Center, the Meshtastic project, the maintainers of the TAK Meshtastic Gateway, the Python Software Foundation, any government agency, Microsoft, or any other company or project mentioned. TAK, ATAK, WinTAK, iTAK, TAK Server, Meshtastic, LoRa, Python, Windows and all other names, logos and trademarks belong to their respective owners and are used here only to identify the products.

### Third-party software and content

- **TAK clients** are not included. Obtain them from the official source and follow the terms you accept there. Do not redistribute them.
- **Meshtastic software and firmware, the TAK Meshtastic Gateway, Python packages, Microsoft build tools and other third-party tools** have their own licenses, maintainers and risks. You are responsible for reviewing and complying with them. This guide does not audit or guarantee them.
- Links point to external sites that the authors do not control.

### Security

This guide covers **basic setup**, not a hardened deployment. It makes no promise that any resulting setup is secure or private. Radio transmissions can be received by others, and anyone with your channel key can read what is sent on it. Protect keys, backups and exported configuration files. You are responsible for securing your systems and the data on them.

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
