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
