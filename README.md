# Bumplog

Logs the iPhone's accelerometer and GPS during a descent. Everything is written to
IndexedDB on the phone. Nothing leaves the device unless you export a file yourself.

## Put it online

1. Make a new public GitHub repo.
2. Drag all six files into it (`index.html`, `sw.js`, `manifest.webmanifest`, the three PNGs).
   Keep them at the root, not in a folder.
3. Settings, Pages, Source: "Deploy from a branch", branch `main`, folder `/ (root)`.
4. Wait a minute, then open the `https://<you>.github.io/<repo>/` URL on the 6s in Safari.

HTTPS is required for both sensors, which is why Pages works and a plain file does not.

## Set the phone up once

- Settings, Safari, Motion & Orientation Access: **on**.
- Settings, Display & Brightness, Auto-Lock: **Never**. iOS suspends JavaScript the instant
  the screen sleeps, so a locked screen means a dead log.
- Share, Add to Home Screen. Launch it from there, not from Safari.
- Open the app. The trace should start moving as soon as you pick the phone up. If the chip
  at the top says "Motion: tap to allow", tap it and accept the prompt.
- Open it once on wifi so the offline copy is stored, then it runs with no signal.

## If there are no motion readings

Open the **Setup** tab. It shows the live x, y, z values and the reading rate, so you can
see where the chain breaks:

- *Readings per second is 0 and no prompt ever appeared* — the prompt was answered
  "Don't Allow" at some point. iOS remembers that and will not ask again, and the Settings
  toggle does not override it. Reset with Settings, Safari, Clear History and Website Data,
  then reopen the app and tap Allow motion access.
- *Secure connection says no* — sensors are blocked on plain HTTP. Use the Pages URL.
- *Launched from home screen says no* — permission granted in Safari does not always carry
  over to the home screen copy, and vice versa. Grant it again in whichever one you use.
- *Readings arrive but the live reading is empty* — events are firing without values, which
  on iOS means the permission is half granted. Same reset as above.

## What a ride shows

Tap any row on the Rides tab. Three charts, all drawn from the samples on the phone:

- **Movement** — the grey band is the full swing of the accelerometer across the whole run,
  one column per pixel of screen. The orange edges are the typical energy in each second, so
  a wide grey band with narrow orange edges means a few big hits on an otherwise calm
  stretch, and both widening together means sustained chatter.
- **Speed and height** — speed in orange, ground height as the shaded area behind it.
- **Track** — the shape of the run, each segment coloured by how rough it was there. Green
  dot is the start. Brighter orange is rougher trail.

No map tiles are fetched, so all of this works with no signal.

## Deleting rides

- One ride: open it and tap Delete, or tap Edit on the Rides tab for a delete button on
  every row.
- Everything: Setup, Delete all rides. Asks twice, then erases the lot.

The figure in the top right is how much of the phone the app is currently using.

## Updating the app

Change the files on GitHub, **then bump `CACHE` in `sw.js`** (`bumplog-v2` to `v3`). The
phone only notices a new version if `sw.js` itself changed, so editing `index.html` alone
will never reach the device. Bump `VERSION` in `index.html` too, so you can confirm on the
phone which build is running.

On the phone: Setup, Check for update, on wifi. If something new is there the button turns
orange and says Restart to finish the update. If an update gets stuck, Re-download the app
files clears the stored copy and fetches everything again. Neither touches your rides.

## Reading the exports

`*-accel.csv` — `t_ms,ax,ay,az`. Milliseconds since the ride started, then acceleration in
m/s squared **with gravity included**, in the phone's own frame. Roughly 60 rows per second.

`*-gps.csv` — `t_ms,epoch_ms,lat,lon,alt_m,acc_m,speed_ms,heading_deg`. Blank where iOS gave
no value. Roughly 1 row per second.

`*.gpx` — the track, for anything that reads GPX.

`*.json` — ride summary plus both streams in one file.

The roughness number is the RMS of |acceleration| after a slow high-pass, in g. Taking the
magnitude makes it independent of how the phone is mounted, so two runs compare even if you
rotated the phone in the mount between them.
