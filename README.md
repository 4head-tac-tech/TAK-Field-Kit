# TAK + Mesh Network Guides

Guide 5 also explains how to use the most updated version of atak and meshtastic to run a tak server through the meshtastic app if you dont want to setup a tak server on a Raspberry PI

Plain-language, step-by-step guides for building a small private **TAK** (Team Awareness Kit) network at home: your own TAK Server on a Raspberry Pi, the **ATAK / WinTAK / iTAK** clients, offline maps, and a **Meshtastic** LoRa radio mesh that works with no cell or internet.

Every guide includes a **common mistakes and corrections** table, a **troubleshooting** table, a **final checklist**, and a **legal notice, license and disclaimer**. All addresses, names and keys are generic placeholders such as `<PI_IP>`, `<username>` and `<PSK>`.

> **Use at your own risk. No warranty.** These are unofficial, community-written guides. They are not affiliated with or endorsed by the TAK Product Center, the Meshtastic project, or anyone else mentioned. Parts were drafted with an AI assistant and tested on a limited set of hardware and software versions. Read [DISCLAIMER](#disclaimer) before you follow anything here.

## Guides

Suggested reading order:

| # | Guide | What it covers |
|---|---|---|
| 1 | [TAK Server on a Raspberry Pi 5](docs/01-TAK-Server-on-Raspberry-Pi-5.md) | Flash Raspberry Pi OS (Bookworm), install TAK Server 5.x with the community installer, admin certificate, first user, fixed IP |
| 2 | [Installing and Setting Up WinTAK, ATAK and iTAK](docs/02-TAK-Clients-Install-and-Basic-Setup.md) | Where to get each client, install, permissions, callsign, connecting to a server |
| 3 | [User Profiles and Connecting Clients](docs/03-TAK-Server-User-Profiles-and-Connecting-Clients.md) | Create accounts, groups, the enrollment package, onboarding people, revoking access |
| 4 | [MOBAC Offline Maps for ATAK, iTAK and WinTAK](docs/04-MOBAC-Offline-Maps-for-ATAK.md) | Build offline map atlases with Mobile Atlas Creator and load them on every client, plus a map-provider terms warning |
| 5 | [Adding Meshtastic to WinTAK, ATAK and iTAK](docs/05-Meshtastic-with-WinTAK-ATAK-iTAK.md) | Overview of the radio bridge for all three clients, ATAK options A and B, version matching |
| 6 | [Meshtastic Node to WinTAK Gateway (Windows)](docs/06-Meshtastic-Node-to-WinTAK-Gateway-Windows.md) | Hands-on companion to guide 5: Python, gateway install errors and fixes, private channel, testing |

**Which path do I want?**

- *I want a server and phones talking to it:* guides 1 → 2 → 3.
- *I need maps without a signal:* guide 4.
- *I need radio with no network at all:* guide 5, then 6 if you use WinTAK.

## How confident are these guides?

Be honest with yourself about what has been tested. Each guide says so in its own "versions used" or "confirmed / not confirmed" notes, but in short:

- **Tested on one setup:** TAK Server 5.8 on a Raspberry Pi 5 (Bookworm), ATAK-CIV 5.6.0.12, and a Meshtastic node on 2.7.x firmware. Chat between WinTAK and phones on the mesh worked in both directions.
- **Not confirmed (so test it yourself):** relaying mesh data to a TAK Server, and parts of the iTAK, WinTAK and MOBAC loading steps. Menu wording and file locations vary between versions.
- **Always** trust the official documentation for your exact version over these guides.

## Conventions

Placeholders are written in angle brackets. Replace them with your own values and **never publish your real ones**.

| Placeholder | Meaning |
|---|---|
| `<PI_IP>` / `<SERVER_IP>` | Your TAK Server's address on your network |
| `<NODE_IP>` / `<PC_IP>` | A Meshtastic node's / a Windows PC's LAN address |
| `<username>` / `<tak_user>` / `<password>` | Accounts and passwords you choose |
| `<CHANNEL_NAME>` / `<PSK>` | Your private Meshtastic channel name and key |
| `<CALLSIGN>` | The name you appear as on the TAK map |

## What is deliberately not here

- **No TAK software.** TAK Server and the TAK clients come from [tak.gov](https://tak.gov), Google Play or the App Store under their own terms. Do not add them to this repository.
- **No keys, certificates or backups.** The [.gitignore](.gitignore) blocks the usual file types (`.p12`, `.pem`, `enrollmentDP.zip`, Meshtastic config backups, map atlases). Check before you commit.
- **No map tiles.** Many tile providers forbid bulk downloading and redistribution. See guide 4.

## Contributing

Corrections are welcome, especially fixes for steps that have changed in newer versions. See [CONTRIBUTING.md](CONTRIBUTING.md). For security-relevant mistakes in a guide, see [SECURITY.md](SECURITY.md).

## License

- **Guide text:** [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE-DOCS.md)
- **Code, scripts and configuration snippets:** [MIT License](LICENSE)

Third-party software and content mentioned in the guides keep their own licenses.

## Disclaimer

This is general boilerplate, not legal advice. The same notice appears at the end of every guide.

- **No warranty; no liability.** The Materials are provided "as is" and "as available." The authors and contributors are not liable for data loss, hardware damage, downtime, security incidents, exposure of locations or data, or any other harm.
- **AI-assisted and version-specific.** Steps may be wrong, incomplete or out of date. Verify against official documentation and test on non-critical equipment first.
- **Commands can be destructive.** Several steps use `sudo`, flash storage, change network settings or run third-party scripts as root. Back up first and read every command.
- **Not affiliated; trademarks.** TAK, ATAK, WinTAK, iTAK, TAK Server, Meshtastic, LoRa, Raspberry Pi, Android, iOS, Windows and other names belong to their owners and are used only to identify products.
- **Privacy and consent.** A TAK network and a radio mesh share **live location**. Get the informed consent of everyone whose position is shared, and follow privacy, radio, telecommunications and export-control law where you live.
- **Radio rules.** Operating a transmitter is regulated. Set the correct region and follow the rules for your area.
- **Security.** These guides cover basic home setups, not hardened deployments. Do not expose a TAK Server to the public internet without proper hardening.
- **Not for safety-critical use.** Do not rely on any of this as your only means of navigation, communication, tracking or emergency response. Carry independent backups.
