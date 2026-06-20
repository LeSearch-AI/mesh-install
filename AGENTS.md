# AGENTS.md — setting up the mesh for a user

You are an AI agent helping a user join one of their machines to **their own private
mesh**. The mesh lets them watch and drive that machine (terminal sessions, coding
agents, stats, SSH, VNC) from their phone/watch/other machines — entirely over their
**Tailscale tailnet**, never the public internet.

Your job is to do this in **as few steps as possible** and **without leaking anything
sensitive**. Read the Security Rules first — they are non-negotiable.

---

## Security rules (do not violate)

1. **Never expose the tailnet.** meshd binds `0.0.0.0`, but the machine must reach it
   *only* over Tailscale — never via a public IP, port-forward, ngrok, or `tailscale
   funnel`. Access is gated by two layers: the Tailscale ACL (who is on the tailnet)
   and a bearer **token** (the API auth). Both must stay in place.
2. **Never write secrets or tailnet identifiers into anything durable.** That means: do
   not print, commit, paste into a public repo/PR/issue, or log:
   - the auth **token**,
   - real **Tailscale IPs** (`100.x.y.z`), MagicDNS names, or machine hostnames.
   Use placeholders in every file and message: `<TOKEN>`, `<tailscale-ip>`,
   `<machine-name>`. If you must show a command that contains the token, tell the user
   to fill it in themselves.
3. **Reuse the user's existing token.** If the user already runs the mesh elsewhere
   (their app has a token), use that same token so the new machine joins the same mesh.
   Only generate a new token for a brand-new mesh.
4. **Prefer Tailscale SSH** for remote setup (key-less, identity-based). Never create,
   copy, or upload SSH private keys to do this.
5. **Run setup on the target machine itself** (locally or via that machine's own shell).
   Do not route a machine's setup through a third box.

---

## Install (one command)

Run this **on the machine being added**. Pick the token per rule 3.

```sh
curl -fsSL https://github.com/LeSearch-AI/mesh-install/releases/latest/download/install.sh | sh -s -- --token <TOKEN>
```

That single command, on **macOS / Linux / WSL** (arm64 or x86_64):

- detects OS + arch, installs `bun` if missing,
- installs meshd (control plane), rmux-bridge (live terminal), and hook tools under
  `~/.mesh`,
- registers a **reboot-persistent service** (launchd on macOS; systemd `--user` + linger
  on Linux/WSL; tmux fallback otherwise) that restarts on crash and after reboot,
- prints the machine's Tailscale IP and the mesh URLs.

Component control: `--only meshd`, or `--without bridge`. Full flags: `--help`.

## Verify

```sh
curl -fsSL https://github.com/LeSearch-AI/mesh-install/releases/latest/download/install.sh | sh -s -- --list
```

Expect `service: meshd (launchd|systemd)` and a present `token`. Then confirm health
**from the machine itself** (loopback, so no token/IP leaks to logs):

```sh
curl -fsS -H "Authorization: Bearer <TOKEN>" http://127.0.0.1:8899/health
```

A `{"ok":true,...}` response means meshd is live. Ports: meshd `8899` (control plane:
stats, sessions, events, usage, tailnet, kb), rmux-bridge `7820` (live terminal), VNC
`6080` (screen). All reached only over the tailnet.

## Connect it in the app

In the MeshWatch app, add a machine with: a **name**, its **Tailscale IP**
(`tailscale ip -4` on that box — the user reads it on-device, you don't record it), and
the **same token**. The app polls the machine over the tailnet; SSH and VNC open through
the same private path.

## Uninstall

```sh
curl -fsSL https://github.com/LeSearch-AI/mesh-install/releases/latest/download/install.sh | sh -s -- --uninstall --purge
```

`--uninstall` stops + removes the service and components; `--purge` also drops the token
and `~/.mesh`.

---

## Troubleshooting (fast path)

- **App can't see the machine** → meshd is down or Tailscale is off. Re-run `--list`; if
  no service, re-run install. Confirm Tailscale is up (`tailscale status`).
- **`token rejected`** → the machine's token differs from the app's. Re-install with the
  app's token (rule 3).
- **Survives reboot?** → yes, by design. If not, the install fell back to tmux (no
  launchd/systemd) — check `--list`; on Linux ensure `loginctl show-user "$USER" -p
  Linger` is `Linger=yes`.
- **Multiple machines** → run the one install command on each, all with the **same
  token**, so they form one mesh.

## Not yet (roadmap — do not improvise these)

These are intended but not built; don't hand-roll them per machine:

- a single common port fronting meshd + bridge + SSH + VNC,
- isolated **agent user-groups** (scoped Unix users so an agent only sees part of a box),
- a **resource + token-budget view** (per-machine memory/CPU and per-provider agent usage
  limits, to route a task to the right machine/agent).

If a user asks for these, say they're on the roadmap rather than improvising an
insecure or one-off version.
