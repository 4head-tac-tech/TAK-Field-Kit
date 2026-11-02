# Contributing

Thanks for helping keep these guides accurate. Software versions change often, so
corrections are very valuable.

## Good contributions

- A step, menu name, file path or command that no longer matches a newer version
  (say which version you tested).
- A mistake that cost you time, with the fix. These go in the "common mistakes"
  tables.
- A "not confirmed" item that you tested, with what you saw.
- Clearer wording, typo fixes, broken links.

## Rules for every change

1. **Keep it generic.** Never include real IP addresses, hostnames, usernames,
   passwords, channel names or keys, certificate contents, device serial numbers,
   phone numbers, email addresses, home or work locations, or screenshots that
   show any of these. Use the placeholders already in the guides
   (`<PI_IP>`, `<username>`, `<PSK>`, and so on).
2. **Say what you tested.** Name the software, firmware and OS versions. Mark
   anything you did not test as unconfirmed.
3. **Keep the legal notice.** Every guide ends with the legal notice, license and
   disclaimer. Do not remove or weaken it.
4. **Keep the structure.** Guides keep their "common mistakes and corrections"
   and "troubleshooting" tables and a final checklist. Add rows rather than new
   sections where you can.
5. **No TAK software, keys or tiles.** Do not add TAK Server or client binaries,
   certificates, enrollment packages, Meshtastic config exports or map atlases.
   The `.gitignore` blocks most of these. Do not work around it.
6. **Respect licensing.** By contributing you agree your text is licensed under
   CC BY 4.0 and your code or snippets under the MIT License.
7. **No unlawful use.** Do not add content meant to track people without consent,
   evade radio or telecommunications rules, or break a service's terms.

## Before you open a pull request

- [ ] I searched my changes for personal and network details and found none.
- [ ] I stated the versions I tested.
- [ ] Relative links between guides still work.
- [ ] The legal notice is intact.
