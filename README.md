# PTC-40 Control

A web controller for **Datavideo PTC-40** PTZ cameras, plus **PTZOptics**, **Avkans** and other VISCA over IP cameras, for 1–8 cameras mixed freely.
It runs on your laptop, and you control the cameras from any browser: the same laptop, a tablet, or a phone on the same network.

```
Browser (laptop / tablet / phone)  ──HTTP──▶  server.js (Node.js on the laptop)  ──DVIP TCP/UDP 5002──▶  PTC-40 cameras
```

A browser can't open raw TCP/UDP sockets, so a small Node.js server does the camera talking. The server has **no dependencies**: no `npm install`, and no internet connection needed at the venue.

> **Installing for the first time? Open `INSTALL.html`** (double-click it). It is the step-by-step guide: Node.js, firewall, network plan, cameras, vMix tally, tablets, autostart and troubleshooting.

## Quick start

1. Install **Node.js LTS** (v18 or newer) from https://nodejs.org.
2. Copy this folder to the laptop.
3. Start it:
   - **Windows:** double-click `start-windows.bat`
   - **Mac / Linux:** double-click `start-mac-linux.command` (or run `node server.js`)
4. Your browser opens at **http://localhost:8080**. Click **Discover** to find the cameras, or use **Add by IP**.

The console prints a second address (e.g. `http://192.168.1.20:8080`). Open that on a tablet to use it as a touch controller.
The first time you start it, allow Node.js through the Windows Firewall (Private networks) so tablets and discovery work.

## Features

| Area | What you get |
|---|---|
| **Pan / tilt** | Analog on-screen joystick (mouse or touch). Speed scales with how far you push. Speed limit slider. |
| **Zoom** | Spring-return T/W rocker with variable speed. Live zoom position read-out. |
| **Focus** | AF / MF, variable-speed near/far rocker, one-push AF. |
| **Presets** | 16 buttons per camera (4–64 in Settings). **Recall**, **Store** (arms for one store, then returns to Recall so you can't overwrite by accident), **Rename** (or right-click any preset). The last recalled preset is highlighted. Names are saved on the server, so every device sees them. |
| **Exposure** | Mode (Auto / Manual / Shutter / Iris / Bright) with direct sliders for iris, shutter, gain, brightness and exposure compensation, plus backlight compensation and WDR. Only the sliders that apply to the current mode are shown. |
| **Colour** | White balance (Auto / Indoor / Outdoor / One-push with trigger / ATW / Manual with red and blue gain). Colour correction: saturation (60–200 %) and hue. |
| **Picture** | Sharpness, noise reduction, gamma (0–4), defog, black & white, flip, mirror, freeze, power / standby. |
| **Image presets** | 8 built-in looks (Full auto, Daylight, Tungsten, Backlit, Low light, Vivid, Natural, B&W). **Save current** stores a camera's complete image setup under a name. Apply it to one camera, or to **all cameras**. **Copy this camera to all** matches every camera to the one you're on. |
| **Read-back** | Every image value is read from the camera and kept in sync across all open browsers. Settings the camera rejects are greyed out. **Read camera** reloads everything. |
| **vMix tally** | Connects to the vMix TCP API (port 8099). Each camera is linked to a vMix input (by input key, so reordering is safe). The tab shows PGM/PVW, and a red ON AIR bar appears above the controls. The camera's own tally light is driven over VISCA (command configurable). Optional confirmation before recalling a preset on the on-air camera. Optional auto-select of the camera that goes to Preview. |
| **Camera brands** | Per camera **Control protocol**: Datavideo DVIP (TCP 5002), VISCA over IP TCP (5678) or UDP (1259) for PTZOptics/Avkans, Sony VISCA over IP (UDP 52381). |
| **Live video** | Optional. Each camera's RTSP stream is shown with ~0.2–0.5 s delay through [go2rtc](https://github.com/AlexxIT/go2rtc) (RTSP → WebRTC, no re-encoding). Views: selected camera, 2×2 grid of all cameras (click to take control), or off. Tally colours around the picture. Storing a preset saves a thumbnail of the shot on its button. Install with `get-video-windows.bat`. |
| **Multi-camera** | Tabs with live connection status (green / amber / red) and colour coding. Automatic reconnect. |
| **Discovery & network** | DVIP broadcast finds every Datavideo device on the LAN. You can also change DHCP, IP, mask, gateway and host name, and reboot. |
| **Keyboard** | See below. |
| **Gamepad / USB joystick** | Xbox/PlayStation pads work out of the box. Map other joysticks' axes in **Settings**, which has a live axis meter. |
| **Safety** | If a browser freezes, loses Wi-Fi or is closed mid-move, the server stops the camera within 0.6 s. **Space** or **Esc** stops everything. |
| **Diagnostics** | VISCA log drawer at the bottom shows every byte sent and received, and lets you send raw VISCA. |

### Keyboard

| Key | Action |
|---|---|
| `← ↑ ↓ →` / `W A S D` | Pan / tilt (hold `Shift` for slow) |
| `Q` / `E` | Zoom wide / tele |
| `Z` / `X` | Focus near / far |
| `F` | One-push autofocus |
| `1`…`9`, `0` | Preset 1–10 (stores when Store mode is armed) |
| `Shift`+`1`…`4`, `[` `]` | Select camera |
| `H` | Home |
| `Space` / `Esc` | Stop all motion |

### Gamepad (standard / Xbox layout)

| Control | Action |
|---|---|
| Left stick | Pan / tilt |
| Right stick ↕ | Zoom |
| LT / RT | Focus near / far |
| A B X Y | Presets 1–4 |
| D-pad ↑ → ↓ ← | Presets 5–8 |
| LB / RB | Previous / next camera |
| View / Menu | One-push AF / Home |
| L3 (click left stick) | Slow mode on/off |

## HTTP API (vMix scripts, Companion, Stream Deck)

Cameras can be addressed by their position (`1`, `2`, …) or by id. GET variants exist so you can fire them from a vMix shortcut, Bitfocus Companion or a Stream Deck "website" button.

```
GET  /api/cam/1/preset/3/recall          recall preset 3 on camera 1
GET  /api/cam/1/home                     home
GET  /api/cam/1/stop                     stop all motion
POST /api/cam/1/preset/3/store           store preset 3
POST /api/cam/1/move   {"pan":-1..1,"tilt":-1..1}   (repeat every <600 ms while moving)
POST /api/cam/1/zoom   {"v":-1..1}
POST /api/cam/1/focus  {"v":-1..1}
POST /api/cam/1/set    {"param":"colorGain","value":6}   any image parameter (see lib/visca.js)
POST /api/cam/1/cmd    {"cmd":"wbOnePushTrigger"}
GET  /api/cam/1/look/b-daylight          apply an image preset
POST /api/cam/1/match-all                copy camera 1's image to all cameras
GET  /api/vmix                           vMix connection, inputs and tally
GET  /api/video                          live-video engine status
POST /api/cam/1/raw    {"hex":"81 01 06 04 FF"}
GET  /api/state        all cameras, presets, settings
GET  /api/events       live updates (Server-Sent Events)
GET  /api/discover     DVIP network scan
```

**Moving cameras from vMix:** call these URLs from a vMix Script (VB.NET), for example `Dim w As New System.Net.WebClient : w.DownloadString("http://127.0.0.1:8080/api/cam/1/preset/3/recall")`, and bind it to a shortcut with `ScriptStart`.

## Configuration

Everything is saved in `data/config.json`: cameras, preset names, image presets, vMix and tally settings. Back it up to keep your show setup. Gamepad axis mapping is saved in each browser.

Environment variables:

| Variable | Default | |
|---|---|---|
| `PORT` | `8080` | Web UI port. Don't use 8088 or 8099 (vMix). |
| `HOST` | `0.0.0.0` | Use `127.0.0.1` to allow only this laptop. |
| `CONFIG` | `data/config.json` | Config file location |
| `GO2RTC` | `bin/go2rtc(.exe)` | Path to an existing go2rtc binary |

## Testing without a camera

```bash
node tools/mock-camera.js                                  # fake PTC-40 on TCP+UDP 5002
node tools/mock-camera.js --port 5003 --no-udp --name Cam2 # second fake camera
node tools/mock-vmix.js                                    # fake vMix TCP API on 8099, cuts every 4 s
node tools/mock-camera.js --proto visca-tcp --port 5678    # PTZOptics/Avkans-style camera (also visca-udp, sony-udp)
```
Then add `127.0.0.1` (port 5002 / 5003) as cameras, and enable vMix tally with host `127.0.0.1`.

## Troubleshooting

- **Discover finds nothing.** The laptop and cameras must be on the same subnet. Allow UDP 5002 in the firewall. Some managed switches block broadcasts, so add the camera by IP instead.
- **Camera tab stays red.** Ping the camera's IP. Make sure no other controller (e.g. a hardware RMC) is holding the DVIP connection. The Discover list shows how many controllers each camera has connected.
- **Preset numbers are off by one compared with the RMC/other software.** Settings → *Preset 1 sends VISCA number* → switch between 0 and 1.
- **Camera is mounted upside down.** Settings → invert pan / tilt, and enable Flip in the Image panel.
- **An image control does nothing.** Not every VISCA command is implemented on every model. Open the VISCA log: a `Syntax error` or `Command not executable` reply means the camera doesn't support it in its current mode (e.g. iris only works in Manual or Iris priority).

## Files

```
server.js              HTTP server, REST API, motion watchdog
lib/dvip.js            DVIP TCP connection (framing, queue, reconnect) + UDP discovery/network config
lib/visca.js           VISCA command and inquiry builders
lib/vmix.js            vMix TCP API client (tally subscription + input list)
lib/video.js           Starts/configures go2rtc for live video (bin/go2rtc[.exe])
lib/config.js          JSON config persistence
public/                Web UI (index.html, app.css, app.js), no build step, no external libraries
tools/mock-camera.js   Fake camera for testing
tools/mock-vmix.js     Fake vMix for testing tally
INSTALL.html           Step-by-step install guide (also at http://localhost:8080/install)
start-windows.bat      Start the controller
allow-firewall-windows.bat  Firewall rules for tablets/phones and discovery (asks for admin)
autostart-windows.bat  Start at Windows log-in (run again to turn off)
get-video-windows.bat  Downloads go2rtc v1.9.9 into bin/ to enable live video
```

## About the PTC-40 command set

The PTC-40 isn't named in Datavideo's DVIP guide (V1.92), but it uses the same DVIP transport as the PTC-140/150 listed there. Camera commands use the standard Sony VISCA set that Datavideo PTZ cameras implement. Every command is listed in `lib/visca.js` and is easy to adjust. If Datavideo publishes a PTC-40-specific VISCA list with different values, the ranges in `DIRECT` (and `PARAMS` in `public/app.js`) are the only places to change.
