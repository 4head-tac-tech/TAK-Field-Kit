# TAK Server: Creating User Profiles and Connecting Others — Guide

A start-to-finish walkthrough for creating user accounts on an existing **TAK Server 5.x** (such as one built with the *TAK Server on a Raspberry Pi 5* guide) and getting other people connected with **ATAK** (Android), plus notes for **iTAK** and **WinTAK**.

> **Use at your own risk. No warranty.** This guide is an unofficial, community-written resource provided "as is." It is not affiliated with or endorsed by the TAK Product Center. Read the full **[Legal notice, license and disclaimer](#legal-notice-license-and-disclaimer)** at the end before you follow it. Everyone you connect to your server shares data (including live location) with it, so get their informed consent first.

**Assumes:** your TAK Server is already installed and running, certificate enrollment is enabled, and you can reach the Admin UI. If not, do that guide first.

---

## Placeholders used in this guide

Replace these with your own values. Never publish your real values in screenshots, forum posts or shared copies of this guide.

| Placeholder | Meaning |
|---|---|
| `<PI_IP>` or `<SERVER_IP>` | The server's fixed address on your network |
| `<admin_user>` | Your Linux/SSH login on the server |
| `<tak_user>` | A TAK account you are creating for a person |
| `<password>` | A strong password for that account |
| `<GROUP>` | A TAK group (channel) name |
| `<server_name>` | The friendly name shown in ATAK for the server |

---

## 1. How it works (read this first)

Four things are involved. Most mistakes come from mixing them up.

| Piece | What it is | Who has it |
|---|---|---|
| **TAK user account** | A username and password stored on the server | One per person |
| **Client certificate** | The file that proves a device is allowed to connect. Created automatically during enrollment | One per device |
| **Enrollment data package** (`enrollmentDP.zip`) | A small zip that tells the app the server address and port, and carries the server's trust certificate. It contains **no personal password** | The same file can be given to every user |
| **Group (channel)** | A label that controls who sees whose data | Optional; users default to a shared group |

**The flow:**
1. You create a username and password for the person on the server.
2. You give them the enrollment package and their username/password.
3. They import the package into ATAK and sign in when prompted.
4. ATAK requests its own certificate from the server (port **8446**), then connects on port **8089**.

Because the package holds no credentials, the safe pattern is: **one shared package, one account per person.**

## 2. Before you start: checklist

- [ ] Server shows `active (running)` (`sudo systemctl status takserver`).
- [ ] The server has a **fixed IP** (reservation or static). If it changes, every package breaks.
- [ ] Certificate enrollment was enabled during install, so `enrollmentDP.zip` exists.
- [ ] Ports **8089** and **8446** are allowed by the firewall (`sudo ufw status`).
- [ ] The person's phone is on the **same network** as the server (LAN-only build).
- [ ] You have a password manager ready to record each account.
- [ ] You have told each person what the server collects and shared it with them (location, callsign, chat, markers) and they agree.
- [ ] You know the app version they will use (ATAK-CIV is assumed here).

## 3. Plan your accounts

Decide these once. Changing them later is annoying.

- **Naming scheme.** Short, lowercase, no spaces, e.g. `alpha01`, `jsmith`. Avoid anything that reveals sensitive real-world details if you ever share screenshots.
- **One account per person.** Do not share one login between people. Shared logins make it impossible to tell users apart or remove one person.
- **Admin vs normal.** Normal users can connect and use the map. Admin accounts can manage the server. Give admin only to people who need it, and keep admin credentials separate from everyday credentials.
- **Password rule.** The server requires **15+ characters** with upper case, lower case, a digit and a special character. Generate them with a password manager.
- **Record keeping.** Store username, password, issue date and device in your password manager. Do not keep them in a plain text file on the server.

## 4. Create a user profile

You can use either method. Both create the same kind of account.

### Option A: command line (over SSH)

```
sudo java -jar /opt/tak/utils/UserManager.jar usermod -p '<password>' <tak_user>
```

- File name is `UserManager.jar` with a capital **U**.
- Wrap the password in **single quotes** so special characters (`!`, `$`, `#`) are not interpreted by the shell.
- This creates a **normal (non-admin)** user.

To create or change an account and place it in a group at the same time:
```
sudo java -jar /opt/tak/utils/UserManager.jar usermod -p '<password>' -g "<GROUP>" <tak_user>
```

To change a password later, run the same `usermod -p` command again for that username.

> Option flags can differ between TAK Server versions. Run `sudo java -jar /opt/tak/utils/UserManager.jar` with no arguments to print the usage for your version, and trust that over this guide.

### Option B: Admin UI (browser)

1. In Firefox (with your admin certificate imported), open `https://<SERVER_IP>:8443`.
2. Go to **Administrative → Manage Users** (the exact menu wording can vary slightly by version).
3. Choose **Add User**, enter the username and password, and select any groups.
4. Save, and confirm the account appears in the list.

The Admin UI is also where you disable or delete accounts and see who is connected.

### Admin accounts

Only create these when needed, and never hand out the admin certificate (`webadmin.p12`) to regular users. The admin certificate gives full control of the server.

## 5. Groups (optional)

By default everyone shares one group and sees each other. Use groups when you want separation, for example a "Team A" and a "Team B" that should not see each other's positions.

- Create or assign groups in the Admin UI when adding a user, or with `-g "<GROUP>"` on the command line.
- A user only sees data from groups they belong to.
- Start simple. Add groups only after the basic setup works for everyone.
- If someone cannot see another user, check group membership first.

## 6. Get and check the enrollment package

Find recent packages the installer created:
```
sudo find / -name "*.zip" -mmin -120 -not -path "/proc/*" 2>/dev/null | head
```
Use **`enrollmentDP.zip`**. You do not need the QUIC variant. (If you installed it longer ago, widen the time window or search by name: `sudo find / -name enrollmentDP.zip 2>/dev/null`.)

Copy it to your PC (run on the PC, not in the SSH window):
```
scp <admin_user>@<SERVER_IP>:~/enrollmentDP.zip .
```

**Verify it before handing it out.** It must point at the right address and port:
```
unzip -p ~/enrollmentDP.zip config.pref | grep -i connect
```
Expect your `<SERVER_IP>` with port `8089`. If it shows a wrong or old address, do not distribute it; fix the address or regenerate the package first.

**Treat the package as private.** It holds no passwords, but it reveals your server's address and trust certificate. Share it directly with the people you intend to connect, not on public channels.

## 7. Connect a person (ATAK on Android)

Give each person: the `enrollmentDP.zip`, their **username**, their **password**, and the rule that their phone must be on the server's network. Send credentials through a different channel than the zip, and never in a group chat.

On their phone:

1. **Get the file onto the phone** (USB, a private cloud link, a messaging app).
2. Open **ATAK**.
3. **Menu → Settings → Import Manager** (the wording can vary), then **Import** and pick **Local SD**.
4. Select `enrollmentDP.zip`.
5. When prompted, enter the **username and password** you gave them. ATAK enrolls and receives its certificate automatically.
6. Wait a few seconds. The server indicator at the top of ATAK should turn green/connected.

**Set their identity** (so you can tell users apart on the map):
- **Menu → Settings → Callsign and Device Preferences** (wording varies).
- Set **Callsign**, **Team color** and **Role**.
- Choose callsigns that follow a simple scheme, e.g. `ALPHA-1`.

Also ask them to keep location permission on, and to turn off battery optimisation for ATAK, otherwise Android may put it to sleep and drop the connection.

## 8. Other clients (iTAK, WinTAK)

The same idea applies: import the package, then sign in.

- **iTAK (iPhone/iPad):** open the zip with iTAK (share/"Open in" from Files). It prompts for the credentials.
- **WinTAK (Windows):** import the data package through the app's import feature, then sign in.

Menu names differ between versions, so check each app's current documentation. Make sure the client version is compatible with your server version.

## 9. Verify on the server side

- Open `https://<SERVER_IP>:8443/Marti/metrics/index.html`. **Clients Connected** should increase by one for each new device.
- In the Admin UI, check the user list and any connection/dashboard views.
- Check the logs if a device will not connect:
  ```
  sudo tail -n 100 /opt/tak/logs/takserver-messaging.log
  ```
  (Log file names can vary by version.)

## 10. Quick onboarding checklist per person

Copy this for each new user:

- [ ] Account created, password recorded in the password manager
- [ ] Person understands and agrees to what the server shares
- [ ] `enrollmentDP.zip` sent (address verified first)
- [ ] Credentials sent on a separate channel
- [ ] Phone on the right network
- [ ] Package imported and enrolled successfully
- [ ] Callsign, team and role set
- [ ] Appears on the server metrics/Admin UI
- [ ] Test: they appear on your map, and you appear on theirs
- [ ] Test: send a chat message and a marker

## 11. Changing, removing and revoking access

**Change a password:** rerun `UserManager.jar usermod -p '<new password>' <tak_user>`, or edit the user in the Admin UI. The person then signs in again with the new password.

**Remove a user (someone should no longer have access):**
1. Delete or disable the account in the Admin UI (**Manage Users**).
2. **Revoke their device certificate** so an already-enrolled phone cannot keep connecting. In the Admin UI use the certificate management page; on the command line, the certificate scripts in `/opt/tak/certs/` include revocation helpers in many versions. Check your version's documentation.
3. Restart or reload the server if your version requires it for revocation to take effect.
4. Confirm they no longer appear under connected clients.

> Deleting the account alone may not disconnect an already-enrolled device. Always revoke the certificate as well.

**Lost or stolen phone:** revoke that device's certificate right away.

## 12. Connecting people who are not on your network

This guide covers a LAN-only build. A person away from your network cannot reach the server by default. Do **not** fix this by forwarding ports on your router to the internet.

Safer options to research and test before relying on them:
- A VPN or private mesh overlay that gives remote devices a private address on your network.
- Proper firewall rules, hardened server configuration and a decision about certificate lifetimes.

Whatever you choose, the enrollment package must point to an address those devices can actually reach. If the address changes, regenerate and redistribute the package.

---

## Common mistakes and corrections

| Mistake | What happens | Correction |
|---|---|---|
| Handing out `webadmin.p12` or admin credentials to regular users | They get full control of the server | Give only the enrollment package plus a normal account. Keep admin cert to yourself |
| One shared login for everyone | Cannot tell users apart; cannot remove one person | One account per person |
| Putting the password in the same message as the zip | Anyone who sees the message has both | Send credentials separately |
| Password under 15 characters or missing a character class | `usermod` rejects it | 15+ characters with upper, lower, digit and special character |
| Password with `!` or `$` typed without quotes | Shell mangles it; wrong password is set | Use single quotes: `'<password>'` |
| Lower-case `usermanager.jar` | `Unable to access jarfile` | File name is `UserManager.jar` |
| Forgetting `sudo` | Permission errors | Run with `sudo` |
| Server IP changed after packages were distributed | ATAK shows connecting but never connects | Give the server a fixed IP and regenerate the package if it changes |
| Never verifying `config.pref` before sharing | Everyone gets a broken package | Run the `unzip -p ... | grep -i connect` check first |
| Importing the QUIC package | Connection issues | Use `enrollmentDP.zip` |
| Phone on cellular or a different Wi-Fi | Cannot reach the server | Same network for LAN-only builds |
| Port 8446 blocked | Enrollment prompt fails or hangs | Allow 8446 and 8089 in `ufw` |
| Typing the username with different case | Login fails | Usernames are case-sensitive in practice; copy exactly |
| Importing a **new** package on top of an old, broken setup without cleaning up | Stale entries conflict | Remove the old server entry in ATAK's network settings, then import again |
| Deleting an account but not revoking the certificate | Old device may stay connected | Revoke the certificate too |
| Android battery optimisation left on | Connection drops when the screen is off | Exempt ATAK from battery optimisation |
| Location permission set to "while using" | Position stops updating in the background | Allow location "all the time" |
| Posting screenshots with passwords, IPs or certificate details | Exposes your server | Blur or crop before sharing |
| Testing only one phone | Problems show up only when a second device connects | Test with at least two devices |
| Skipping the consent conversation | People are tracked without understanding it | Explain what is shared and get agreement first |

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| Import works but no sign-in prompt | Wrong package (not `enrollmentDP.zip`), or enrollment was not enabled during install. Check that port 8446 is listening: `sudo ss -tlnp | grep 8446` |
| Sign-in rejected | Wrong username or password, or password was changed. Reset it with `usermod -p` and retry |
| Prompt appears but enrollment fails | Port 8446 blocked, phone not on the network, or server still starting. Check `ufw` and `systemctl status takserver` |
| Connected, but shows no other users | Users are in different groups, or the others are not connected yet. Check group membership |
| Connects briefly then drops | Battery optimisation, weak Wi-Fi, or time mismatch on the phone. Turn on automatic date and time |
| Certificate or SSL errors | Phone clock wrong, or an old server certificate left from an earlier install. Remove the old server entry and re-import |
| Worked yesterday, not today | Server IP changed, server did not restart cleanly, or certificate expired. Check the IP, then `systemctl status takserver` |
| "Clients Connected" stays at 0 | Check the server is running, the package address is right, and the logs |
| Cannot find the package | `sudo find / -name enrollmentDP.zip 2>/dev/null` |

---

## Credits and links

- TAK Server and ATAK: TAK Product Center — https://tak.gov
- Community installer: https://github.com/myTeckNet/installTAK

*Written for TAK Server 5.x. Menu names and options change between versions. Verify against the official documentation.*

---

## Legal notice, license and disclaimer

**This is general boilerplate, not legal advice.** If you need certainty about your obligations, ask a qualified lawyer.

### Use at your own risk; no warranty

This guide, and any scripts, configuration files or other code distributed with it (together, the "Materials"), are provided **"as is" and "as available," without warranty of any kind**, express or implied. That includes, without limitation, any warranty of merchantability, fitness for a particular purpose, accuracy, completeness, security, reliability or non-infringement.

To the fullest extent permitted by law, the authors and contributors are **not liable** for any claim, damages or other liability, whether in contract, tort or otherwise, arising from or connected with the Materials or their use. This includes data loss, corrupted or wiped storage, hardware damage, downtime, security incidents, exposure of data or locations, financial loss, injury, or any other harm. You follow this guide entirely at your own risk.

### Accuracy and AI assistance

Parts of this guide were drafted with the help of an AI assistant and may contain errors, omissions or outdated steps. Software, command options and menu names change between versions. Always check against the official documentation for TAK Server, ATAK, iTAK, WinTAK and any other software you use, and test on non-critical systems first.

### Commands can be destructive

Several steps use `sudo`, change accounts and certificates, or alter server access. A mistake can lock people out or lock you out of the server. Back up first, and read every command before you run it.

### Not affiliated; trademarks

This is an unofficial, independent community guide. It is **not affiliated with, endorsed by or sponsored by** the TAK Product Center, any government agency, Raspberry Pi Ltd, or any other company or project mentioned. TAK, ATAK, WinTAK, iTAK, TAK Server, Raspberry Pi, Android, iOS, Windows, Firefox and all other names, logos and trademarks belong to their respective owners and are used here only to identify the products.

### Third-party software and content

- **TAK Server and TAK clients** are not included. Obtain them from the official source (tak.gov) and follow the terms you accept there. Do not redistribute them.
- **Third-party tools** have their own licenses, maintainers and risks. You are responsible for reviewing and complying with them.
- Links point to external sites that the authors do not control.

### Security

This guide covers a **LAN-only build**, not a hardened production deployment. It makes no promise that the resulting server is secure. Do not expose the server to the public internet without proper hardening. Keep certificates, passwords, enrollment packages and backups protected. You are responsible for securing your systems and the data on them.

### Privacy, consent and the law

A TAK Server can collect and share the **live location and other data** of everyone connected to it. You are responsible for complying with all laws and rules that apply to you, including privacy and data-protection law, consent from the people who use your server, radio and telecommunications rules, employment or organisational policies, and export-control and other legal requirements for the software and hardware you use. Do not use these Materials for any unlawful purpose, including to track anyone without their knowledge and consent.

### Not for safety-critical use

Do not rely on a home-built server, a phone or any software described here as your **only** means of navigation, communication, tracking or emergency response. Equipment fails, batteries die, networks drop and data can be wrong or stale. Carry independent backups, test your setup in advance, and use sound judgment, especially outdoors or in any situation where a failure could put someone at risk.

### License

Unless stated otherwise, and subject to the disclaimers above:

- **The text of this guide** is licensed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license.
- **Any code, scripts or configuration files** distributed with this guide are licensed under the **MIT License**.

Copyright (c) 2026 4head-tac-tech. Third-party software and content keep their own licenses. By contributing, you agree that your contribution is licensed on the same terms. The warranty disclaimer and limitation of liability above apply to everyone who uses, copies, modifies or redistributes the Materials.

### Reporting problems

Found a mistake, a security issue or an outdated step? Please open an issue or a pull request in the project repository: https://github.com/4head-tac-tech/TAK-Field-Kit.
