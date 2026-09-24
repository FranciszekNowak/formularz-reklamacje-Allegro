# ADR-001: Operator workstations and how Claude Code reaches the VPS

Date: 2026-09-24
Status: Accepted (enrollment of `Laptop_FN` in progress, see "Enrollment state")

## Context

`CLAUDE.md` grants Claude Code standing permission to SSH into the production VPS
(`franciszek@185.25.149.174`) for diagnostics and for operating this repo's own Compose
project. That permission is only usable from a machine whose key the server already
accepts, and this repo does not own the server's access model: the VPS host, its SSH
configuration and the list of authorised keys belong to `Haxe_agent_system`, which
decided them in its ADR-010 (`architecture/decisions/010-vps-hardening-and-access.md`).

Three facts make enrollment a gate rather than a formality:

- **Password authentication is disabled** on the server. ADR-010 originally left it on as
  a fallback and its dated update of 2026-06-01 turned it off once both desktops were
  keyed. Access is key-only. A machine with no key therefore cannot enroll itself; the
  public key has to be appended by an already-keyed machine, or through the Cyberfolks
  (Solus) VNC console.
- **Only two machines were keyed** under ADR-010: the home desktop and the office desktop.
- **Work on this repo now happens from unkeyed machines.** `STATUS.md` (2026-09-23)
  recorded the marketing desktop `komputer-marketing` as unkeyed. On 2026-09-24 a third
  unkeyed machine was measured: this laptop, `Laptop_FN` (Windows 11 Home 10.0.26200,
  user `franc`), which has no `~/.ssh` directory at all, so no key and no known-hosts
  entry.

Two further things were measured on `Laptop_FN` on 2026-09-24, because they change the
enrollment procedure written in `STATUS.md`:

- **The `ssh-agent` service is `Stopped` and `StartType: Disabled`,** and Claude Code's
  shell does not run elevated, so the service cannot be enabled from inside a session.
  Enabling it requires an Administrator PowerShell window driven by the operator.
- **Two different SSH clients are installed, and the wrong one wins by default.**
  Windows OpenSSH 9.5p2 lives at `C:\WINDOWS\System32\OpenSSH\ssh.exe`; Git Bash ships
  its own at `/usr/bin/ssh` and shadows the Windows one on `PATH` inside a Bash session,
  where `SSH_AUTH_SOCK` is empty. Only the Windows client talks to the Windows
  `ssh-agent` service. A key loaded with `ssh-add` is therefore invisible to Git Bash,
  which would fall back to reading the key file and block on a passphrase prompt that a
  non-interactive tool call cannot answer.

## Decision

1. **Every machine this repo is operated from is enrolled under agent-system ADR-010,
   never by a change made from this repository.** This repo may request enrollment and
   record that it happened; the authorised-keys change, the server's SSH configuration and
   the canonical list of keyed machines stay owned by `Haxe_agent_system`. Each enrollment
   is written back there as a dated amendment to ADR-010, and this repo's dependency on
   that ADR is pinned in `contracts/agent-system.md`.
2. **`Laptop_FN` is enrolled as a third operator workstation** for `franciszek`, with its
   own ed25519 keypair, generated on the laptop so the private key never moves between
   machines. The same applies to `komputer-marketing` when it is keyed.
3. **Claude Code drives SSH through the Windows OpenSSH client, not Git Bash** — in this
   project's tooling that means the PowerShell tool rather than the Bash tool. This
   follows `CLAUDE.md`'s existing preference for Windows-native commands, and here it is
   load-bearing rather than stylistic: it is the only client that reaches the agent, so it
   is the only one that can authenticate without a human typing the passphrase.
4. **The key is held by the Windows `ssh-agent` service, set to start automatically,** so
   that the passphrase is typed once per boot by the operator and Claude Code can use the
   key unattended afterwards. The passphrase itself is never stored in this repo, never
   passed on a command line, and never typed into a Claude Code tool call.
5. **The server's host key is pinned in this repo** as
   `SHA256:Zl0i+LIcg4GfNrkREfguhyIIwDax+ilmms1MHsAWhBQ` (ED25519), and a new machine's
   first connection is accepted only against a fingerprint confirmed from an already
   trusted connection — `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` run on the
   server itself. A fingerprint read with `ssh-keyscan` from the machine being enrolled
   confirms nothing: it comes from the same unauthenticated channel as the connection it
   is supposed to be validating.

## Enrollment state

| Machine | Keyed | Recorded in agent-system ADR-010 |
|---|---|---|
| Home desktop | yes | yes, 2026-06-01 update |
| Office desktop | yes | yes, 2026-06-01 update |
| `Laptop_FN` | in progress, 2026-09-24 | amendment still to be proposed from `Haxe_agent_system` |
| `komputer-marketing` | no | no |

## Consequences

- Claude Code can operate the VPS from this laptop once enrollment completes, under the
  standing authorisation already in `CLAUDE.md` and its guardrails: this repo's own
  Compose project and nginx file only, no `.env` edits without per-task confirmation,
  `sudo nginx -t` before every reload.
- The number of keys that can reach a production host grows with each workstation, and
  every one of them is a Windows desktop holding a passphrase-protected private key. That
  is accepted for a single-operator system; the mitigation is that keys are per-machine,
  so losing one machine means revoking one line of `~/.ssh/authorized_keys` rather than
  rotating a shared secret. Revocation is an ADR-010 operation, performed from
  `Haxe_agent_system`.
- Because password authentication is off, the two keyed desktops and the VNC console are
  the only ways to enroll anything else. If both desktops became unavailable before
  `Laptop_FN` is keyed, enrollment would have to go through the Solus console.
- Sessions running in Git Bash cannot reach the server unattended even after enrollment.
  This is a permanent property of the split client setup, not a transient state; the
  discipline in decision 3 is what keeps it from surfacing as an unexplained
  passphrase-prompt hang.
- `STATUS.md` section 4 covers the same procedure for `komputer-marketing`. This ADR
  supersedes its step 5 only in that the amendment to ADR-010 must name every machine
  enrolled since 2026-06-01, not just the marketing desktop.
