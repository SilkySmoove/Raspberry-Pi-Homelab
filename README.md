# Raspberry-Pi-Homelab
My journey building a Raspberry Pi homelab from scratch

## Day 1 and 2- First Boot & Headless SSH Access

**Date:** July 9-10, 2026
**Goal:** Flash Raspberry Pi OS Lite (64-bit), boot headless, and log in over SSH from a Windows PC.

### What I did
- Flashed Raspberry Pi OS Lite (64-bit) using Raspberry Pi Imager
- Configured hostname, username, and SSH before first boot
- Found the PI's IP via the router's devices list (Also accessible via ISP app)
- Connected over SSH from Windows PowerShell

### Problems I hit (and how I solved them)
- **No microSD card reader** - laptop had no SD slot; last minute run to acquire a MicroSD Adapter. *Lesson: AI can provide a list of possible expenses and a plan, but there is always an unknown. Must always verify*
- **Password kept failing** - the flashed password wasn't being accepted over SSH. Resulted in numerous attempts to failed access attempted and reflashing of the microSD. Root Cause: when you SSH into your Raspberry Pi, the command is `SSH <username>@<ip address>`
- **"REMOTE HOST IDENTIFICATION HAS CHANGED"** — each re-flash gave the Pi a new host key while Windows remembered the old one. Fixed with `ssh-keygen -R <ip>`.

### Commands learned
- `ssh piserver@<ip>` — open a secure remote shell to the Pi
- `ssh-keygen -R <ip>` — remove a stale saved host key
- `passwd` — change the current user's password
- `hostname -I` — show the Pi's IP address

### Deliverable ✅
A running, headless Raspberry Pi server, accessible over SSH from my Windows PC.

Day 3 and 4 - Network-Wide DNS Routing & Multi-Device Debugging

Date: July 11-12, 2026
Goal: Point the whole home network's DNS at Pi-hole so ad-blocking applies to every device, not just the Pi itself.

What I did


Logged into the ISP router's admin panel using the sticker credentials
Found DNS settings under Basic → DNS, set DNS Obtain to Manual, DNS1 to the Pi's static IP
Verified Pi-hole was resolving correctly with nslookup before troubleshooting the dashboard
Rebooted the router to force all devices onto the new DNS via DHCP
Cross-referenced Pi-hole's Clients list against the router's Connected Devices list (by IP and MAC) to find which devices weren't routing through Pi-hole
Diagnosed and fixed iCloud Private Relay, a stale Bitdefender VPN/DNS profile, and an IPv6 DNS bypass — three separate issues all blocking one iPhone from ever reaching Pi-hole


Problems I hit (and how I solved them)



Pi-hole dashboard showed 0 queries despite correct config - Turned out to just need a refresh and real browsing traffic. Confirmed the DNS path itself was fine with nslookup google.com <pi-ip>, which bypasses the router entirely.
One iPhone never appeared in Pi-hole at all - Root cause was two stacked issues on the same phone: a leftover Bitdefender VPN/DNS proxy profile still active despite the app's "protection" toggle being off, and the router's IPv6 DNS still set to Auto, letting the phone bypass Pi-hole entirely over IPv6. Fixed by deleting the VPN profile under Settings → General → VPN & Device Management, and manually setting IPv6 DNS1 on the router to Pi-hole's IPv6 address.
A previously-working phone seemed to "break" - False alarm — it had simply cached a DNS answer (visited google.com right before testing) and had no reason to make a fresh query. Confirmed it was fine by testing an uncached, random domain instead.


Commands learned


nslookup google.com <pi-ip> — test a DNS server directly, bypassing the router/OS DNS config
ipconfig /flushdns / /release / /renew — force Windows to drop and reacquire network settings, including DNS
ipconfig /all — show the DNS server currently assigned to a Windows network adapter


Deliverable ✅

Every known device on the network — including the two stubborn edge cases — routing DNS through Pi-hole, confirmed live in the Query Log.


Day 5 - Blocklists, Streaming Ads, and a DHCP Reservation Scare

Date: July 17, 2026
Goal: Improve blocking coverage beyond the default list, and harden the setup against future data loss or IP drift.

What I did


Added HaGeZi Multi Normal and HaGeZi Threat Intelligence Feeds as additional blocklists, alongside the default StevenBlack list
Ran pihole -g to rebuild the blocklist database against the new lists
Learned why DNS-based blocking can't catch every ad — YouTube, TikTok, and server-side-inserted streaming ads (a category Crave partly falls into) are stitched directly into the content stream, so there's no separate ad domain to block
Backed up the full Pi-hole config via Settings → Teleporter → Export
Attempted to add a DHCP reservation on the router, tying the Pi's static IP to its MAC address, as a second layer of protection beyond the OS-level static IP


Problems I hit (and how I solved them)


Crave/TikTok ads still getting through after adding blocklists - Confirmed this is a structural limitation of DNS blocking (server-side ad insertion), not a config issue — no blocklist can fix it without breaking the app/stream entirely.
Entire network lost internet after adding the router DHCP reservation - Every device failed with DNS_PROBE_FINISHED_BAD_CONFIG, including devices set to Automatic DNS — the tell that the issue was upstream (the DNS server address itself unreachable) rather than a per-device setting. Fixed with a full router power cycle (30 sec unplug, 2-3 min to rejoin the network).


Commands learned


pihole -g — rebuild Pi-hole's blocklist database after adding/removing adlists
ping <ip> — test raw reachability to a device, independent of DNS
ping 8.8.8.8 — test raw internet connectivity while bypassing DNS entirely, useful for isolating DNS issues from real outages


Deliverable ✅

Broader ad/tracker coverage via HaGeZi, a portable Teleporter backup of the full config, and a documented recovery process for router-level DNS outages.
