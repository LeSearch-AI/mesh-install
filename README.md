# mesh-install

One curl command to install, start, and uninstall the **mesh stack** — `meshd`
(stats / sessions / events daemon), `rmux-bridge` (live terminal stream), and the
agent hook tools — on any **macOS, Linux, or WSL** machine (arm64 / x86_64).

Services are registered with a real supervisor so they **survive reboot**:
launchd on macOS, systemd `--user` (with linger) on Linux/WSL, tmux fallback
otherwise. Built on Bun + TypeScript — no Node, no heavyweight runtime.

## Install

```sh
curl -fsSL https://github.com/LeSearch-AI/mesh-install/releases/latest/download/install.sh | sh
```

With a shared token (use the same one your MeshWatch app is configured with):

```sh
curl -fsSL https://github.com/LeSearch-AI/mesh-install/releases/latest/download/install.sh | sh -s -- --token YOURTOKEN
```

Only meshd, or everything except the bridge:

```sh
... | sh -s -- --only meshd --token YOURTOKEN
... | sh -s -- --without bridge --token YOURTOKEN
```

## Uninstall

```sh
curl -fsSL https://github.com/LeSearch-AI/mesh-install/releases/latest/download/install.sh | sh -s -- --uninstall
# also remove the token + ~/.mesh:
... | sh -s -- --uninstall --purge
```

## Status

```sh
curl -fsSL https://github.com/LeSearch-AI/mesh-install/releases/latest/download/install.sh | sh -s -- --list
```

See [`install/README.md`](install/README.md) for all flags, environment
variables, the hook tools, and how to rebuild the release bundle.
