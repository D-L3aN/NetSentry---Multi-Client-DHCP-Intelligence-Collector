# NetSentry---Multi-Client-DHCP-Intelligence-Collector
Continuous DHCP client reconnaissance for the Shark Jack Display. Hosts a DHCP server and captures hostname, IP, MAC, MAC vendor (OUI), and connection time for every client that connects — until you flip the switch back to Arming Mode. Writes a timestamped log to `/root/loot/`.
# NetSentry — Multi-Client DHCP Intelligence Collector

Continuous DHCP client reconnaissance for the Shark Jack Display. Hosts a
DHCP server and captures hostname, IP, MAC, MAC vendor (OUI), and
connection time for every client that connects — until you flip the
switch back to Arming Mode. Writes a timestamped log to `/root/loot/`.

- **Author:** D-L3aN
- **Version:** 1.0
- **Category:** recon
- **Target:** Shark Jack Display
- **Props:** Inspired by "Simple Client Info" by Hak5 (Darren Kitchen)

---

## Authorised Use Only

This payload is intended for **authorised security testing, red team
engagements, and internal IT audits only**. Deploying it against networks
you do not own or do not have explicit written permission to test is
illegal.

NetSentry is **passive and non-destructive**. It does not:

- Modify or disrupt network traffic beyond normal DHCP
- Scan ports, fingerprint services, or exploit vulnerabilities
- Send any data off-device (log stays local in `/root/loot/`)
- Authenticate to any resource

---

## What it collects

| Field | Source |
|---|---|
| Hostname | DHCP option 12 (hostname) |
| IP address | Assigned DHCP lease |
| MAC address | DHCP client hardware address |
| Vendor | OUI lookup from MAC prefix (IEEE database) |
| Timestamp | Local system time at capture |

Each client is logged with a numbered entry, and a session summary is
written when the payload stops.

---

## How it works

1. **DHCP_SERVER netmode** — the Shark Jack becomes the DHCP server on
   the connected segment.
2. **Continuous WAIT_FOR_CLIENT loop** — each DHCP request is captured.
3. **Environment variables** `$CLIENT_HOSTNAME`, `$CLIENT_IP`, and
   `$CLIENT_MAC` are populated by the Shark Jack framework for each
   client.
4. **OUI lookup** — the first 3 bytes of the MAC are matched against
   `/usr/share/ieee-data/oui.txt` if present (standard on most Linux
   distributions; installed by default on Shark Jack firmware).
5. **Log written** to `/root/loot/netsentry_<timestamp>.log`.
6. **Switch check** — flipping the physical switch to Arming Mode
   causes the loop to exit cleanly, writes the summary, and displays
   the final count.

---

## Usage

1. Copy `payload.txt` to `/root/payload/` on the Shark Jack Display.
2. Flip the switch to **Attack Mode**.
3. Plug the Shark Jack into the target switch or client port.
4. Watch the on-screen counter increment as clients connect.
5. When done, flip the switch back to **Arming Mode**.
6. Retrieve the log via SSH:

```bash
ssh root@172.16.24.1
cat /root/loot/netsentry_*.log
```

---

## Sample output

```
==============================================
 NetSentry Recon Log
 Started: 2025-05-02 14:22:07
 Device:  shark
==============================================

----------------------------------------------
Client #1
  Time     : 2025-05-02 14:22:31
  Hostname : DESKTOP-A1B2C3
  IP       : 172.16.24.100
  MAC      : AA:BB:CC:11:22:33
  Vendor   : Intel Corporate
----------------------------------------------

----------------------------------------------
Client #2
  Time     : 2025-05-02 14:22:47
  Hostname : MacBook-Pro
  IP       : 172.16.24.101
  MAC      : DD:EE:FF:44:55:66
  Vendor   : Apple, Inc.
----------------------------------------------

==============================================
 Session Summary
 Ended:   2025-05-02 14:25:03
 Clients: 2
 Log:     /root/loot/netsentry_20250502_142207.log
==============================================
```

---

## Configuration

Edit the top of `payload.txt`:

| Variable | Default | Purpose |
|---|---|---|
| `LOOT_DIR` | `/root/loot` | Where logs are stored |
| `SCREEN_REFRESH` | `2` | Seconds between screen updates |
| `VENDOR_LOOKUP` | `true` | Set `false` to skip OUI lookups (faster on slow links) |

---

## LED feedback

| LED | Meaning |
|---|---|
| `LED SETUP` | Initializing, creating log file |
| `LED ATTACK` | DHCP server active, listening for clients |
| `LED G SOLID` | Momentary — a client was just captured |
| `LED FINISH` | Collection stopped, log written |

---

## Tested on

- Shark Jack Display, firmware 1.3.0 — **Pass**

---

## Known limitations

- **DHCP option 12 is optional.** Some clients (particularly iOS and
  certain hardened Windows configurations) do not send a hostname in
  their DHCP request. In those cases the hostname field shows
  `unknown`, but IP and MAC are still captured.
- **OUI database required for vendor lookup.** If
  `/usr/share/ieee-data/oui.txt` is missing on your firmware build,
  vendor will show `unknown`. Set `VENDOR_LOOKUP=false` to skip.
- **Single segment only.** DHCP servers only see clients on the same
  broadcast domain. For routed networks, the Shark Jack must be
  connected to the target VLAN.
- **Static IP clients are invisible.** Clients configured with static
  IPs never send DHCP requests, so they will not appear in the log.

---

## Differences from Simple Client Info

The original **Simple Client Info** payload captures a single client
and displays three lines to the screen. NetSentry extends it with:

1. **Continuous multi-client capture** — every client that connects is
   logged, not just the first.
2. **Persistent loot log** — timestamped file in `/root/loot/`, not
   just transient screen output.
3. **MAC vendor lookup** — identifies hardware vendor from OUI.
4. **Session summary** — client count and duration written at exit.
5. **Graceful stop** — flip switch to Arming Mode to end collection
   cleanly and write the summary.
6. **Live counter on screen** — see how many clients captured at a glance.
7. **Structured log format** — numbered entries, consistent fields,
   easy to parse with `grep` or `awk`.

---
