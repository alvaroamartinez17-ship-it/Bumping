# Bumplog

Logs the iPhone's accelerometer and GPS during a descent. Everything is written to
IndexedDB on the phone. Nothing leaves the device unless you export a file yourself.

## Put it online

1. Make a new public GitHub repo.
2. Drag all six files into it (`index.html`, `sw.js`, `manifest.webmanifest`, the three PNGs).
   Keep them at the root, not in a folder.
3. Settings → Pages → Source: "Deploy from a branch", branch `main`, folder `/ (root)`.
4. Wait a minute, then open the `https://<you>.github.io/<repo>/` URL on the 6s in Safari.

HTTPS is required for both sensors, which is why Pages works and a plain file does not.

## Set the phone up once

- Settings → Safari → Motion & Orientation Access: **on**.
- Settings → Display & Brightness → Auto-Lock: **Never**. iOS suspends JavaScript the
  instant the screen sleeps, so a locked screen means a dead log. The app warns you if it
  notices this happened.
- Share → Add to Home Screen. Launch it from there, not from Safari.
- First tap on "Start recording" triggers the motion permission prompt. Accept it.
- Open it once on wifi so the service worker caches everything, then it runs with no signal.

## Reading the exports

`*-accel.csv` — `t_ms,ax,ay,az`. Milliseconds since the ride started, then acceleration in
m/s² **with gravity included**, in the phone's own frame. Roughly 60 rows per second.

`*-gps.csv` — `t_ms,epoch_ms,lat,lon,alt_m,acc_m,speed_ms,heading_deg`. Blank where iOS
gave no value. Roughly 1 row per second.

`*.gpx` — the track, for anything that reads GPX.

`*.json` — ride summary plus both streams in one file.

The roughness number the app shows is the RMS of |acceleration| after a slow high-pass,
expressed in g. Taking the magnitude makes it independent of how the phone is mounted, so
two runs are comparable even if you rotated the phone in the mount between them.

## Changing the app later

Edit `index.html`, and **bump `CACHE` in `sw.js`** (`bumplog-v1` → `v2`). Without that the
phone keeps serving the cached old version and your change never appears.
