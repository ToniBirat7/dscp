# DSCP

Pool the idle NVIDIA GPUs of a department (lab workstations, students' laptops) so the group can fine-tune and serve models no single machine can hold.

This repository holds the **signed releases and the installer**.
The source opens later; the binaries here are what every machine of a pool runs.

## What it does

- One machine holds the pool (the coordinator); every other machine lends its card when its owner allows.
- A job is split across machines by layers, and several copies train together and average every few minutes (DiLoCo), so a laptop that closes its lid costs its own group a few minutes, not the run.
- A job sleeps when the room empties at night and wakes when machines return.
- The pool answers prompts through an OpenAI-compatible API and a chat page.
- Credits record who lent what and who used it.

## Your machine stays yours

- **Pause, at once:** `dscp pause`, or the tray app's button; `dscp resume` to lend again.
- It steps aside by itself on battery, when it gets hot, or when a full-screen game or video starts.
- It runs as you, never as root, and keeps everything under one folder (`~/.dscp` by default).
- What machines send each other is encrypted; the coordinator's address is pinned to the pool's key.

## Requirements

- Linux on x86_64 with an NVIDIA card (driver 525.60 or newer) for each machine that trains.
- Any Linux machine, even without a GPU, to hold the pool.
- No Python, Docker or Node: the binary builds its own runtime the first time it needs one.
- Windows and macOS: not yet. Apple GPUs do not train here yet.

## Install

```bash
curl -fsSL https://github.com/ToniBirat7/dscp/releases/latest/download/install.sh | sh
```

The installer checks the release's Ed25519 signature before it installs anything.
The desktop app for Debian and Ubuntu is the `.deb` on the [releases page](https://github.com/ToniBirat7/dscp/releases).

## Start a pool

On the machine that will hold it:

```bash
dscp service install --coordinator   # keeps the pool running
dscp coordinator link                # signs your browser in as the pool's owner
dscp coordinator invite              # a link (and QR code) for each machine that joins
```

On each machine that lends its card:

```bash
dscp join <the link>
dscp service install                 # or `dscp agent` to try it in the foreground
```

`dscp doctor` says what is missing on a machine; `dscp status` says what it is doing.
Jobs are submitted from the pool's web page.

## Remove it

```bash
dscp service uninstall
rm -rf ~/.dscp ~/.local/bin/dscp ~/.local/bin/dscp-worker
```

## Verify a download by hand

`SHA256SUMS` names every file of a release by digest, and `SHA256SUMS.sig` is an Ed25519 signature over it.
The public key is in `install.sh`, between the `release-key` markers.

```bash
sed -n "/^PUBKEY='/,/END PUBLIC KEY/p" install.sh | sed "s/^PUBKEY='//; s/'$//" > release-key.pub
openssl pkeyutl -verify -pubin -inkey release-key.pub -rawin -in SHA256SUMS -sigfile SHA256SUMS.sig
sha256sum -c SHA256SUMS --ignore-missing
```

## Status

Used by one pool today.
Its first week-long unattended run did not yet pass every check; the causes are fixed, and the next run follows.
Expect rough edges, and tell us about them.

## Problems and security

Open an issue here.
For a security problem, use GitHub's private vulnerability reporting on this repository, not a public issue.

## Licence

Apache 2.0.
