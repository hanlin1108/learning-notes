# Mac Mini Headless Setup

> How to set up a Mac Mini once on a TV, then run it headlessly from a MacBook over Screen Sharing.

---

## Table of Contents

1. [Part 1 — Initial Setup on TV](#1-part-1--initial-setup-on-tv)
2. [Part 2 — Prepare for Headless Use](#2-part-2--prepare-for-headless-use)
3. [Part 3 — Move to Bedroom](#3-part-3--move-to-bedroom)
4. [Part 4 — Connect from MacBook](#4-part-4--connect-from-macbook)
5. [Troubleshooting](#5-troubleshooting)
6. [Optional Upgrade](#6-optional-upgrade)

---

## 1. Part 1 — Initial Setup on TV

1. **Connect Mac Mini to TV** using an HDMI cable
2. Plug in power, keyboard, and mouse
3. Switch TV to the correct HDMI input
4. Walk through macOS setup — Apple ID, Wi-Fi, etc.

---

## 2. Part 2 — Prepare for Headless Use

While still connected to the TV, change these three settings:

### Enable Screen Sharing

> System Settings → General → Sharing → toggle on **Screen Sharing**

Write down the address shown (e.g. `192.168.1.xx` or `YourMacMini.local`)

### Turn On Automatic Login

> System Settings → Users & Groups → Automatic Login → select your account

⚠️ If this option is greyed out, you need to turn off FileVault first:

> System Settings → Privacy & Security → FileVault → Turn Off

### Confirm Wi-Fi

> Make sure the Mac Mini is on the **same Wi-Fi network** as your MacBook

---

## 3. Part 3 — Move to Bedroom

1. Shut down the Mac Mini
2. Unplug everything — HDMI, keyboard, mouse
3. Move it to your bedroom
4. Plug in **power only** (and Ethernet if available)
5. Press the power button and **wait ~60 seconds** for it to boot

---

## 4. Part 4 — Connect from MacBook

**Option A — Finder**

1. Open Finder
2. Look in the sidebar under **Network**
3. Click your Mac Mini → click **Share Screen**

**Option B — Spotlight**

1. Press **Cmd + Space**
2. Type **Screen Sharing** and open the app
3. Enter the Mac Mini's address (from Part 2)
4. Click **Connect**

You now have full control of your Mac Mini from your MacBook.

---

## 5. Troubleshooting

| Problem | Fix |
|---|---|
| Can't find Mac Mini on network | Wait a bit longer — it may still be booting. Try entering the IP address directly. |
| Screen Sharing won't connect | Mac Mini may be stuck at login. Reconnect to TV briefly and enable Automatic Login. |
| Laggy or slow | Use 5GHz Wi-Fi on both Macs, or connect Mac Mini via Ethernet. |
| Lost the IP address | On your MacBook, open Terminal and type: `dns-sd -B _rfb._tcp` |

---

## 6. Optional Upgrade

If you use this setup daily and the lag bothers you, consider **Luna Display** (~$130). It's a hardware dongle that makes your MacBook behave like a real monitor — minimal lag, works at the login screen, no extra configuration.
