# claude-usage-fw

Prebuilt firmware **releases** for the Claude usage display on **M5Stack Core 2**
and **M5Stack Fire**. The device shows your Claude.ai usage (session + weekly,
projection graph) on its screen.

This repo is the **public release/OTA channel** — devices on v1.1.0+ check it
automatically and self-update. The source code lives in a separate repo.

> 📥 **Get the latest firmware:** [**Releases**](../../releases/latest)

---

## 1. Which file do I download?

Each release has two kinds of file per board — pick by **how you flash**:

| File | Use | Flash at |
|------|-----|----------|
| `m5stack-<board>-<ver>.bin` | **First flash over USB** (full image) | `0x0` |
| `firmware-<board>-<ver>.bin` | **Update via SD card** (app only) | — (OTA) |

`<board>` = `m5stack-core2` or `m5stack-fire`.
**A blank/new device → use the `m5stack-...` full image** (Step 2).
After it's running, future updates happen **automatically over WiFi** (Step 5).

---

## 2. Flash the firmware (first time, over USB)

Plug the M5 into your computer with a **USB-C data cable** (not charge-only).

### Method A — M5Burner (easiest, no command line)
1. Install **M5Burner**: https://docs.m5stack.com/en/download
2. Open it → the *Burn* / custom-firmware tab → **Burn from a local file**.
3. Select `m5stack-<board>-<ver>.bin`, set address **`0x0`**, baud `921600`, **Burn**.
4. No port showing? Install the USB-UART driver:
   https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers

### Method B — esptool (command line)
```bash
pip install esptool

# find the port: macOS  ls /dev/cu.usbserial-*   Linux  ls /dev/ttyUSB*   Windows  Device Manager -> COMx
esptool.py --chip esp32 --port <PORT> --baud 921600 \
  write_flash -z --flash_mode dio --flash_freq 80m --flash_size 16MB \
  0x0 m5stack-core2-<ver>.bin
```

---

## 3. Get your Claude `sessionKey`

By default the device talks to **claude.ai directly**, so it needs your login
cookie (`sessionKey`).

1. In **Chrome**, log in to **https://claude.ai** (the account you want to show).
2. Open DevTools: **F12** (or Cmd-Opt-I / Ctrl-Shift-I).
3. Go to the **Application** tab -> left sidebar **Cookies** -> **https://claude.ai**.
4. Find the row named **`sessionKey`** and copy its **Value**
   (a long string starting with `sk-ant-sid...`).

> :warning: **Treat `sessionKey` like a password** — it grants access to your
> Claude account. Don't share it or commit it anywhere public. It changes when
> you log out; paste a fresh one if the device says the session is invalid.

---

## 4. Configure the device

Pick **one** way to enter your WiFi + `sessionKey`:

### A) microSD card (recommended)
Put a file named **`claude-usage.json`** at the **root** of a FAT32 microSD card:

```json
{
  "wifiSsid": "YOUR_WIFI",
  "wifiPass": "YOUR_WIFI_PASSWORD",
  "apiDirect": true,
  "claudeSession": "sk-ant-sid...-PASTE-YOURS-HERE",
  "tzOffsetSec": 25200
}
```
Insert the card, power on. `tzOffsetSec` is your timezone in seconds (25200 = UTC+7).

### B) Serial console (no SD card)
Open a serial terminal at **115200 baud** and type:
```
apimode direct
session sk-ant-sid...-PASTE-YOURS-HERE
wifi-add "YOUR_WIFI" "YOUR_WIFI_PASSWORD"
reboot
```

### C) On-screen WiFi portal (no SD card)
On first boot it shows **`ClaudeUsage-setup`**. Join that WiFi from your phone,
open the page, and enter your WiFi (and proxy fields if you use proxy mode).

Once connected it shows your usage. The org is auto-discovered.

> **Prefer not to put your sessionKey on the device?** There's also a **proxy
> mode** (`apimode proxy`) where a small server holds the credential and the
> device only carries an access token — better when handing a device to someone
> else. See the source repo.

---

## 5. Automatic updates (OTA)

Once running **v1.1.0 or newer**, the device checks this repo's latest release
**at boot and every 6 hours** and **updates itself over WiFi** — no cable, no SD.

- Default is auto-install. To only be notified (a tap-to-install banner on the
  Settings page), set `otaAuto` off (serial: `otaauto 0`).
- Check now from serial: `otacheck`.

So you normally flash over USB **once**, then never need a cable again.

---

## Buttons

- **Core 2** (touch): tap left/right of the bottom bar = prev/next page; middle = refresh (or Theme on Settings).
- **Fire**: **BtnA** prev page, **BtnC** next, **BtnB** refresh. On Settings, A/C adjust, B selects.
- **Hold BtnA** (any board) = re-open WiFi setup. **Hold BtnB** on the Projection page = clear its history.

Boards: M5Stack **Core 2** and **Fire** (ESP32, 16 MB).
