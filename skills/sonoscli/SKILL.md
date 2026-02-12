---
name: sonoscli
description: Control Sonos speakers (discover/status/play/volume/group).
homepage: https://sonoscli.sh
metadata:
  {
    "openclaw":
      {
        "emoji": "🔊",
        "requires": { "bins": ["sonos"] },
        "install":
          [
            {
              "id": "go",
              "kind": "go",
              "module": "github.com/steipete/sonoscli/cmd/sonos@latest",
              "bins": ["sonos"],
              "label": "Install sonoscli (go)",
            },
          ],
      },
  }
---

# Sonos CLI

Use `sonos` to control Sonos speakers on the local network.

## When to Use This Skill

**Use when:**

- Controlling Sonos speakers (play, pause, volume)
- Discovering speakers on the local network
- Managing speaker groups or queues
- Searching/playing music via Spotify integration

**Don't use when:**

- Need general music playback (non-Sonos) → use other audio tools
- Need to control non-Sonos smart home devices → use `openhue` or other skills
- Speakers are not on the same network: Sonos requires LAN access

**Success Criteria:**

- Speaker responds to command
- Status output confirms expected state

Quick start

- `sonos discover`
- `sonos status --name "Kitchen"`
- `sonos play|pause|stop --name "Kitchen"`
- `sonos volume set 15 --name "Kitchen"`

Common tasks

- Grouping: `sonos group status|join|unjoin|party|solo`
- Favorites: `sonos favorites list|open`
- Queue: `sonos queue list|play|clear`
- Spotify search (via SMAPI): `sonos smapi search --service "Spotify" --category tracks "query"`

Notes

- If SSDP fails, specify `--ip <speaker-ip>`.
- Spotify Web API search is optional and requires `SPOTIFY_CLIENT_ID/SECRET`.

### Common Pitfalls

**What NOT to do:**

- Forgetting `--name` flag: may target wrong speaker
- Assuming SSDP always works: if it fails, use `--ip <speaker-ip>`
- Using Spotify search without credentials: requires SPOTIFY_CLIENT_ID/SECRET
