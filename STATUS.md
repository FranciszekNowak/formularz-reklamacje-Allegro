# Status & session log — 23 Sept 2026

Handoff note. Read this first when picking the work back up.

---

## 1. Where it stands

The repository was created on 2026-09-23 with conventions copied from
`book-notes-pipeline` and the shared-infrastructure rules from `haxe-flow-metrics`.
Nothing is built or deployed.

## 2. Decided

- **Subdomain `reklamacje.skalus.com`, served from the existing VPS IP
  `185.25.149.174`.** No second IPv4. nginx already serves `agents.skalus.com`,
  `flow.skalus.com` and the book-notes server from that one IP by hostname
  (SNI), and certbot issues one certificate per subdomain. A second IP would be
  a paid Cyberfolks order plus a host network change owned by
  `Haxe_agent_system`, justified only by something like separate IP reputation
  for outgoing mail. Decided by Franciszek, 2026-09-23. To be recorded in the
  ADR that confirms what this repo owns.

## 3. Blockers found on 2026-09-23 (measured, not assumed)

- **DNS points elsewhere.** `reklamacje.skalus.com` already has an explicit A
  record to `5.252.231.210`, whose reverse DNS is `d9.thecamels.org` (TheCamels
  hosting). A random subdomain does not resolve, so this is not a wildcard.
  `skalus.com` itself is on `91.198.146.229`. The zone is at tld.pl
  (`dns1.tld.pl`). Before certbot can issue a certificate, the record must point
  to `185.25.149.174`. First find out who created it and whether anything
  still uses it.
- **SSH from the marketing desktop (`komputer-marketing`) is not set up.** It
  has no `~/.ssh` folder, so it has no key and no known-hosts entry. Only the
  home and office desktops are keyed (agent system ADR-010), and SSH password
  login is disabled. Live host key fingerprint read from here:
  `SHA256:Zl0i+LIcg4GfNrkREfguhyIIwDax+ilmms1MHsAWhBQ` (ED25519). Not yet
  confirmed from a trusted connection.

## 4. Keying the marketing desktop (Franciszek, planned from the home desktop)

1. On the marketing desktop, in a normal PowerShell window (not the `!` prefix,
   which cannot answer prompts):
   `ssh-keygen -t ed25519 -C "franciszek@komputer-marketing"`, with a
   passphrase typed by hand.
2. From the home desktop, append the new `id_ed25519.pub` line with `>>` (never `>`):
   `ssh franciszek@185.25.149.174 "echo '<public key line>' >> ~/.ssh/authorized_keys"`.
   Alternative without a keyed machine: Solus VNC console plus
   `curl -fsS https://github.com/<user>.keys | tee -a ~/.ssh/authorized_keys`.
3. Confirm the host key over the trusted connection:
   `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`. It must match the
   fingerprint above.
4. On the marketing desktop: `ssh franciszek@185.25.149.174 hostname`, accept
   only if the fingerprint matches. Then, in an Administrator PowerShell, run
   `Get-Service ssh-agent | Set-Service -StartupType Automatic; Start-Service ssh-agent`,
   followed by `ssh-add` so Claude can run SSH without the passphrase.
5. Afterwards, propose a dated update to agent system ADR-010 (third keyed
   machine). That file belongs to `Haxe_agent_system`.

## 5. Open decisions

- What the form does, who fills it in, and where a submitted complaint goes
  (fills the TODOs in `CLAUDE.md` "Project context" and "Project goal").
- Build vs. buy for the form itself (see `CLAUDE.md`, the Wardley gate).
- Whether this needs its own container on the VPS at all, or can live entirely
  inside n8n workflows plus a static page.

## 6. Next step

Write the first master plan in `architecture/build-plans/`, starting with Block 0
(memory headroom via `free -h` and `docker stats`, nginx and certbot readiness
for a new vhost, DNS for `reklamacje.skalus.com`).
