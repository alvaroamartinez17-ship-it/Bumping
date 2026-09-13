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

## Comparing runs at the same trail

When a trail holds more than one run, its header gets a **Compare** button. That draws every
run's roughness on one chart and lists their figures side by side: roughness while moving,
hardest hit, vertical and sideways split, top speed, distance, descent, riding time, stops.

The chart plots roughness against **distance covered**, not against time. With time on the
axis, a run that included a stop has its whole trace squashed: in a test where two riders
took the same 600 m trail and one stopped to move a rock, the same rock garden drew 8
percent of the chart wide on one run and 3 percent on the other. Against distance both drew
it at 43 to 52 percent. Where two lines rise together, that is the same section of trail.

If none of the runs has usable GPS the chart falls back to time and says so.

## Riding time

A run usually includes time spent stopped at the top or sorting yourself out at the bottom,
and those seconds read close to zero roughness, which drags the average down. One real run
spent 64 of its 164 seconds stationary.

So the ride detail reports **roughness while actually moving**, measured only from the
samples taken above 5 km/h, alongside how much of the run that was. Use that figure when
comparing runs. Both go into the exports as `roughness_moving_g` and `moving_time_s`.

Descent is measured the same way. GPS altitude wandered 491 to 464 to 453 to 469 to 443 m
across 64 stationary seconds on one run, and the old filter banked 34 m of that as real
descent, reporting 82 m where the hill was about 50. Height now only accumulates while
moving.

## Surface

Roughness separates surfaces cleanly. From a day of real rides on this phone:

| roughness | what it was |
| --- | --- |
| 0.18 to 0.28 g | street |
| around 0.50 g | gravel, light chatter |
| 0.64 to 0.86 g | woodland trail |

The ride detail names the surface from those bands.

## Mounting

The ride detail shows the ratio of sideways to vertical energy. Shapeless shaking with no
preferred direction gives 1.41, so anything below that means vertical hits dominate. Across
thirteen rides on one day this sat between 0.73 and 1.05 regardless of surface, including a
very smooth street run, so **it does not currently tell you whether the phone is firmly
mounted**. Treat it as raw information until there is data from a known rigid mount to
calibrate it against.

The detail reports the share of readings **over 9 g**, which is simply a count of very hard
hits. An earlier version called this sensor saturation, on the evidence of three rides that
all peaked at 9.64 to 9.65 g. A later day reached 12.14 g, so there is no ceiling anywhere
near there and that conclusion was wrong. Typically 0.2 percent of samples exceed 9 g on a
rough descent.

## Airtime

In free fall every axis reads close to zero, so a collapse in total g is the giveaway. Three
guards keep noise out: the dip has to last at least 180 ms, it has to average under 0.40 g,
and a landing of at least 1.6 g has to follow within 450 ms. Unweighting over a root, a hard
compression and rock-garden chatter all fail at least one of those.

Jumps show up live under the trace while you ride, as yellow marks along the top of the
Movement chart, and as yellow dots on the Track. Rides recorded before this feature existed
get analysed when you open them, so your old runs are covered too.

Hang height is `g * t^2 / 8`, which assumes you land at the same height you took off from.
A drop-off lands lower, takes longer, and so reads high. Treat it as a rough guide; airtime
itself is the honest number.

All six thresholds live in the `AIR` object near the top of the script. If your trail keeps
fooling it, that is the place to adjust:

    ENTER 0.30 g   the dip that might be flight
    EXIT  0.55 g   back on the ground
    MIN_MS 180     shorter than this is a jolt
    MEAN_G 0.40 g  average across the dip
    LAND_G 1.6 g   how hard a landing must be
    GAP_MS 250     ignore a new jump this soon after the last

## Sessions and comparing runs

The Rides tab groups by the day a run happened on. Each day is a **session**, numbered in
the order they occurred, so Session 1 is the first day you ever recorded.

Within a session the runs are grouped by **where they happened**, which is what makes two
runs comparable. Each ride gets an anchor: the median latitude and longitude of its usable
fixes, median rather than mean so a stray tower fix cannot drag it across the valley. Rides
whose anchors fall within 500 m of each other become a **trail**, numbered in the order they
first occurred, and the runs inside a trail are numbered Run 1, Run 2 and so on.

The trail header shows how many runs it holds, their mean roughness and their mean fix
accuracy, so repeated runs of the same trail read against each other directly. Rides with no
usable position land in **Location unknown** at the end.

500 m was chosen against a real day of thirteen rides: it merges two runs whose centres sat
362 m apart and keeps a trail run separate from a street run 900 m away. `TRAIL_RADIUS` at the
top of the script changes it.

The day export carries `# session,N` plus `trail`, `anchor_lat` and `anchor_lon` per ride.

## GPS accuracy gates

iOS reports accuracy in discrete buckets and is pessimistic about it. Calibrated against a
day of thirteen rides on an iPhone 6s:

| what was seen | accuracy reported |
| --- | --- |
| open sky, clean track | 5 to 10 m |
| street riding, track still followed the road exactly | a flat 65 m |
| no satellite lock, position from cell towers | 2000 to 13800 m |

So the thresholds are deliberately loose. Fixes coarser than 300 m are rejected outright as
tower guesses; anything up to 200 m is used for distance and height. A step only counts
towards distance if it is larger than `0.15 * accuracy` and does not imply more than 22 m/s,
which filters jitter without discarding real movement.

An earlier version demanded 25 to 30 m and produced a distance of zero on street rides where
the track was plainly correct.

## Where a ride's track starts

The GPS watch runs before you tap Start, so a fix is ready when you push off. The cost is
that the first fix delivered after Start can be one measured earlier, wherever the app was
opened. Two rules keep it out of the ride:

- a fix whose own timestamp predates the ride is discarded
- the ride's first stored point must be accurate to 25 m or better

The second matters because iOS will re-deliver a stale position with a fresh timestamp, and
the tell is that it is wifi derived and coarse. A bad seed point distorts both the track
shape and the distance total, so the app waits rather than accept one. The GPS chip shows
"GPS settling" while it waits, and the count of discarded fixes goes into the CSV header as
`gps_prestart_fixes_dropped`.

## If GPS stops working

The watch iOS gives a web app is fragile. A timeout under tree cover, the screen sleeping
between runs, or the app being backgrounded can all kill it, sometimes without raising any
error at all. The app now handles that itself:

- any error tears the watch down and builds a new one, backing off from 2 up to 15 seconds
- a watchdog restarts the watch if no fix arrives for 15 seconds while recording, or 45
  seconds while idle
- returning to the app after more than 10 seconds away forces a fresh watch
- a refused permission stops the retries instead of looping forever
- the watch runs from the moment the app opens and is never switched off, so a run never
  waits on a tap; it is rebuilt every 10 minutes while idle to stop iOS degrading it

Setup shows how long ago the last fix arrived and how many restarts have happened. A ride
that needed restarts records the count, shown on its detail screen.

## Runs that never started

Starting the recording and then not setting off for a while is easy to do. A run is thrown
away rather than saved when it covered under 30 m, spent under 5 s moving, **and** its
roughness is below 0.15 g. All three have to be true: a real descent with dead GPS reports
no distance and no moving time, but it is never that quiet, so it is kept.

The app says what happened rather than saving it silently.

## Stops during a run

Stretches below 3 km/h lasting 5 s or more are counted as stops. They are kept out of the
roughness figure and the descent count, and the ride detail reports how many there were and
how long they lasted. Moving a rock out of the trail no longer flattens the run's numbers.

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

Two CSVs, both one flat table, both carrying the ride's metadata as `#` comment lines at
the top. Load with `pandas.read_csv(path, comment="#")`.

`*-seconds.csv` — **the one to send.** One row per second: `t_s, roughness_g, lat, lon,
alt_m, speed_kmh, in_air`. A ten minute ride is 600 rows, about 27 KB. Small enough to
open anywhere, mail to yourself, or paste into a message.

`*-all.csv` — everything, one row per accelerometer sample, about 2.8 MB for ten minutes:

| column | meaning |
| --- | --- |
| `t_ms` | milliseconds since the ride started |
| `ax, ay, az` | acceleration in m/s squared, **gravity included**, phone frame |
| `lin_x, lin_y, lin_z` | the same reading with gravity removed by iOS |
| `rot_alpha, rot_beta, rot_gamma` | gyroscope, degrees per second |
| `lat, lon, alt_m, gps_acc_m, speed_ms, heading_deg` | written only on rows where a new fix arrived |
| `in_air` | 1 while a detected jump is in progress |

The GPS columns are blank on most rows because fixes arrive about once a second while the
accelerometer runs at sixty. Fill them downwards after loading: `df[gps_cols].ffill()`.

The `#` header carries duration, distance, descent, top speed, roughness, the vertical and
lateral split, GPS restarts, and one `# jump,` line per jump with its start, duration,
landing force and hang height.

`*.gpx` — the track, for anything that reads GPX.

`*-summary.json` and `*.json` — the same data as JSON, if you prefer it.

The roughness number is the RMS of |acceleration| after a slow high-pass, in g. Taking the
magnitude makes it independent of how the phone is mounted, so two runs compare even if you
rotated the phone in the mount between them.
