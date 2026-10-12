# Running omdrop outside Omarchy: Ubuntu 26.04 + KDE Plasma notes

Notes from getting `omdrop` working on a PC with an Intel AX201 (iwlwifi), Ubuntu 26.04,
KDE Plasma on Wayland, using [omdrop-owl](https://github.com/t4t5/omdrop-owl) as the radio
backend and [owldrop-indicator](https://github.com/joakee/owldrop-indicator) for the tray.
Nothing here is Ubuntu-specific beyond the package names. Tested 2026-10; omdrop 0.16.1.

## What `omdrop setup` does without Omarchy

Only user-level changes, no sudo: the `airdrop-receiver.service` user unit (written, not
started), a symlink at `~/.local/bin/omdrop`, the double-click handler for `.url`/`.webloc`,
the visible name and a TLS identity in `~/.omdrop/keys`. `plugin_dir` is found through
`manifest.json`, so running it from a plain `git clone` works.

## Arch assumptions that break elsewhere

| Symptom after `omdrop setup` | Cause | Fix |
|---|---|---|
| `tomllib` traceback, "could not save the name" | `python3` on PATH is older than 3.11 (e.g. conda's 3.8) | Put the system python first (see `contrib/ubuntu/omdrop-wrapper`) |
| `vercmp: command not found`, then "backend contract older than 0.7.0" | `vercmp` is a pacman tool | `contrib/ubuntu/vercmp` on PATH |
| "no Wi-Fi interface omdrop-owl can use" | PATH without `/usr/sbin`, so `iw` is not found | Wrapper above, or fix PATH |
| "packages the receiver needs (python-libarchive-c keyutils) are not installed" | `receiver_libs_installed` tests `/usr/lib/libkeyutils.so.1`; Debian multiarch puts it in `/usr/lib/x86_64-linux-gnu/` | This branch: test with `ldconfig -p` instead |

On the keyutils test, do not use `grep -q` in the pipeline: with `set -o pipefail`, `grep -q`
closing early can make `ldconfig` exit 141 and the check fails intermittently. The patch
captures the output in `$(...)` and tests `-n`.

Debian/Ubuntu packages for the receiver: `python3-libarchive-c keyutils` (and for the radio
backend: `python3-zeroconf python3-ifaddr`, see omdrop-owl).

## Symlink vs wrapper

`omdrop setup` replaces `~/.local/bin/omdrop` only if it is missing or points into a plugin
directory, and prints "Left ... alone" otherwise. With the wrapper installed that message
is expected and harmless.

## Caveats

- On an Intel AX201, omdrop-owl runs in *exclusive* mode: Wi-Fi goes off while the machine is
  discoverable. If Wi-Fi is the only connection, turning receiving on cuts the network.
- `omdrop on` was deliberately not run during this setup; everything above was checked with
  `omdrop setup`, `omdrop status --json` and `omdrop-discoverable probe --json`.
