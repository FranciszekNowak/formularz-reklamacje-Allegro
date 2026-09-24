# Contract with Haxe_agent_system

Everything this repo borrows from the agent system, pinned by identifier. If the owning
repo changes any of these, this repo breaks silently — so list each dependency here the
moment it is introduced, with the date and how it was verified.

| Resource | Identifier | Owner | Verified |
|---|---|---|---|
| VPS host | `franciszek@185.25.149.174` | Haxe_agent_system | 2026-09-24: port 22 reachable from `Laptop_FN`; no login attempted (not yet keyed) |
| SSH access model | ADR-010, `architecture/decisions/010-vps-hardening-and-access.md` | Haxe_agent_system | 2026-09-24: read at commit `c15da12`. Key-only — password auth disabled by its 2026-06-01 update. Keyed machines listed there are the home and office desktops |
| VPS SSH host key (ED25519) | `SHA256:Zl0i+LIcg4GfNrkREfguhyIIwDax+ilmms1MHsAWhBQ` | Haxe_agent_system | 2026-09-24 via `ssh-keyscan` from `Laptop_FN`, matching the value `STATUS.md` recorded on 2026-09-23. **Not** confirmed from a trusted connection yet — see ADR-001 |
| Host nginx + certbot | `/etc/nginx/conf.d/` | Haxe_agent_system | — |
| n8n instance | `https://agents.skalus.com` | Haxe_agent_system | — |
