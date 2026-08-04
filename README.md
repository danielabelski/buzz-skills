# Buzz + Hermes Skills

A small, portable collection of [Hermes Agent](https://hermes-agent.nousresearch.com/) skills for operating Hermes through [Buzz](https://github.com/block/buzz): connecting a remote Hermes gateway to a Buzz community and delivering local media as native Buzz attachments.

## Contents

| Skill | Purpose |
| --- | --- |
| [`hermes-in-buzz`](./hermes-in-buzz/) | End-to-end setup and troubleshooting for a dedicated Hermes Agent running as a native Buzz messaging agent. Covers relay connectivity, a separate Nostr identity, owner-only access controls, Buzz CLI installation, NIP-OA attestation, gateway configuration, and live inbound/outbound verification. |
| [`buzz-media-attachments`](./buzz-media-attachments/) | Sends local media files to the active Buzz conversation through the native `buzz messages send --file` flow when generic `MEDIA:/path` delivery is unavailable or unreliable. Includes MP4 fast-start and canonical sanitization workflows plus GIF fallback. |
| [`buzz-self-hosting`](./buzz-self-hosting/) | Guides an agent helping a user stand up and debug a self-hosted Buzz relay via the `deploy/compose` stack. Covers env/secrets setup, closed-relay membership, the loopback-hostname 404 footgun (`localhost` vs `127.0.0.1`), community re-keying and stale desktop state, layer-by-layer agent-response debugging, Windows/Git Bash pitfalls, admin surfaces, backups, and hosted→self-hosted migration limits. |

## Repository layout

```text
buzz-skills/
├── buzz-media-attachments/
│   └── SKILL.md
├── buzz-self-hosting/
│   └── SKILL.md
└── hermes-in-buzz/
    ├── SKILL.md
    └── scripts/
        ├── update_buzz_credentials.py
        └── hermes_buzz_credential_helper.rs
```

`hermes-in-buzz/scripts/update_buzz_credentials.py` is a cross-platform credential updater. It reads keys locally with hidden input, atomically updates the active Hermes profile's `.env`, and never accepts secrets in command-line arguments.

`hermes_buzz_credential_helper.rs` is an optional Rust helper intended to be built from the same pinned `block/buzz` source release as the Buzz CLI. It derives a public agent identity and creates NIP-OA owner attestations without placing private keys in argv.

## Install

Copy one or both skill directories into the active Hermes profile's skills directory, preserving all nested files:

```bash
# Default Hermes home
cp -R hermes-in-buzz buzz-media-attachments buzz-self-hosting ~/.hermes/skills/
```

On Windows, the equivalent default destination is:

```text
%USERPROFILE%\.hermes\skills\
```

If `HERMES_HOME` is set or you use a profile-specific Hermes home, install under that resolved directory instead. Start a new Hermes session after copying so skill discovery refreshes.

For a Git-based install:

```bash
git clone https://github.com/tonbistudio/buzz-skills.git
cp -R buzz-skills/hermes-in-buzz buzz-skills/buzz-media-attachments ~/.hermes/skills/
```

## Usage

### Connect Hermes to Buzz

In a fresh Hermes session:

```text
/hermes-in-buzz
```

Or ask:

> Use the hermes-in-buzz skill to connect this Hermes host to my Buzz community.

The skill defaults to the native Hermes gateway topology:

```text
Buzz Desktop / web client
          │
          ▼
      Buzz relay
          ▲
          │ Nostr WebSocket + Buzz CLI
          │
Remote Hermes gateway
```

The Buzz client and Hermes host connect independently to the relay; the client does not need an SSH connection to the Hermes machine and can be offline while the gateway keeps running.

### Attach a local media file

Ask Hermes to attach a local video, GIF, or other supported media file to the current Buzz chat. The `buzz-media-attachments` skill verifies the source, finds the live channel UUID, invokes the Buzz CLI's native file attachment mechanism, and checks Buzz's returned acceptance result.

For MP4 files rejected by the relay, the skill first diagnoses the relay error, then uses a fast-start remux or canonical metadata-free H.264/AAC re-encode when needed. It can convert short clips to animated GIFs when the user requests an image attachment or canonical MP4 is unsuitable.

## Security and operating principles

- Use a dedicated Buzz agent identity; never reuse a human owner's `nsec` as the agent key.
- Never paste private keys, auth tags, API tokens, or owner credentials into chat.
- Keys are entered locally with hidden input and are never passed through command-line arguments.
- Default to `allow_all_users: false`, an explicit owner allowlist, and `require_mention: true`.
- Write non-secret Hermes settings with `hermes config set`, not by manually editing `config.yaml`.
- Do not report a connection or attachment as successful solely from process state: verify a real relay/CLI result and, for gateway setup, a live inbound and outbound Buzz exchange.
- Load `.env` files with a dotenv parser rather than shell-sourcing them. `BUZZ_AUTH_TAG` may contain JSON that shell parsing can corrupt.

## Requirements

The exact requirements depend on the skill path and target platform, but typically include:

- Hermes Agent with gateway support
- A Buzz relay URL and a dedicated agent identity
- The external `buzz` CLI for outbound messages
- Python for `update_buzz_credentials.py`
- Rust/Cargo only when building Buzz or the optional credential helper from source
- FFmpeg/FFprobe for media inspection, MP4 repair, and GIF conversion

Before setting up a gateway, consult the current Hermes and Buzz documentation. The `hermes-in-buzz` skill treats the official Hermes Buzz integration documentation and the latest stable `block/buzz` release as authoritative when they differ from the skill text.

## License

The `hermes-in-buzz` skill metadata declares the MIT license. Unless a separate license file is added, verify licensing expectations before redistributing the repository as a whole.
