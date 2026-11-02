# Security

These guides describe basic home setups. They are **not** a hardened deployment
and make no promise that the result is secure.

## Reporting a problem in a guide

If a guide contains advice that could weaken someone's security or privacy (for
example, a step that exposes a key, opens a port to the internet, or leaks a
location), please report it.

- For anything that does **not** involve your own secrets, open an issue using
  the "Guide correction" template.
- If your report involves real keys, addresses or locations, **do not post them
  publicly.** Describe the problem in general terms and use GitHub's private
  vulnerability reporting for this repository if it is enabled, or contact the
  repository owner privately.

## If you published a secret by mistake

Deleting a file or commit does not remove it from history or from forks and
caches. Treat anything that was public as compromised and replace it:

- **Meshtastic channel key:** generate a new key and re-share the channel link
  with your group.
- **TAK passwords:** change them. Revoke and reissue affected certificates.
- **Admin certificate (`webadmin.p12`) or its password:** regenerate it, and do
  not reuse the old one.
- **Server address:** consider a new address or additional firewall rules.

## Scope

This repository contains documentation only. Third-party software mentioned in
the guides (TAK Server and clients, Meshtastic, the community installer and
gateway, MOBAC, and others) has its own maintainers and security process. Report
problems in those projects to them.
