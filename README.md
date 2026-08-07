# Health Tracker

A single-file web app for tracking **sleep** (bedtime + deep sleep), **weight**,
and **exercise** (weightlifting day, cardio, other activities) — all in one place,
so you can eyeball whether they move together. No build step, no dependencies, no
server — `index.html` is the whole application. (The repo/file are still named
`sleep-tracker`; rename them if you like, it's cosmetic.)

## What it does

**Sleep**
- **One-button bedtime logging.** Press *Log bedtime now* and it stamps the current
  time against tonight. A timestamp after midnight is filed under the previous
  night, so a 12:40am bedtime counts toward the night it belongs to.
- **Deep sleep**, either imported from Apple Watch or typed in by hand.
- **Two adjustable goals** — a target bedtime (default 22:30) and a minimum deep
  sleep (default 90 min) — drawn as dashed reference lines on both charts.

**Weight**
- One number per day, in kg to 2 decimals. The field pre-fills with your most
  recent entry, and you adjust it three ways: type an exact value, click the
  `−1 / −.1 / +.1 / +1` stepper buttons, or **scroll the mouse wheel / drag up-down
  on the number itself** (hold Shift while scrolling for 0.5kg jumps). A drag has
  to move a few pixels before it "takes over" from a tap, so tapping to type still
  works on touch screens.

**Exercise**
- **Weightlifting**: a two-way toggle for whichever day-split you use (default
  "Day 1" / "Day 2" — rename them in *Goals & settings*, e.g. "Push" / "Pull" or
  "Upper" / "Lower").
- **Cardio**: minutes for the day.
- **Other activity**: a name + minutes, add as many as you did that day (e.g.
  "Tennis 60m" and "Walk 20m" on the same date) — shown as removable chips before
  you save.

**Across all three**
- **Progression at a glance:** nightly/daily values plus a 7-day rolling average
  on the weight and sleep charts, so you read the trend rather than one noisy day.
- **The Weight chart and the Activity strip below it share the same date axis**,
  specifically so you can scan down a given day and see whether a lift day, cardio,
  or another activity lines up with a change in weight or sleep — see "Looking for
  correlation" below.
- Stat tiles for both sleep (average bedtime, nights on target, average deep sleep,
  streak) and weight/exercise (current weight, change over the range, lift days,
  cardio + other-activity totals); a table view with inline editing; light and
  dark themes.

## Looking for correlation between sleep, exercise, and weight

The Weight chart, the Activity strip (Lift / Cardio / Other, one column per day),
the Bedtime chart, and the Deep sleep chart are stacked in that order and all
share the exact same date range and day-to-day alignment. There's deliberately no
overlaid dual-axis chart — plotting weight and deep sleep on one shared axis would
visually imply a correlation coefficient the data doesn't actually contain — but
because every chart lines up on the same x-axis, you can scan straight down a
date and see, at a glance, "logged a lift day here, and slept longer that night,
and weight ticked down two days later." That's the intended way to read it.

Food/calorie intake isn't tracked here (tracked separately, per your own setup) —
adding it later would slot in the same way, as one more chart aligned to this axis.

### About the "deep sleep integration" question

Since sleep, weight, and exercise now live in the *same* app and the same Drive
file, this is handled for free — there's no separate sync to wire up. (If you'd
rather keep them as two separate apps for some reason, it's still possible via
Google's file picker and the same `drive.file` scope, but it's extra plumbing for
no real benefit once they're merged like this.)

## Hosting it on GitHub Pages

1. Go to [github.com/new](https://github.com/new). Name the repo anything (e.g.
   `sleep-tracker`), set it **Public** (GitHub Pages on a free account requires this —
   the data itself isn't in the repo, only the app code), and click **Create repository**.
2. Upload the files: on the repo's page, click **Add file → Upload files**, drag in
   `index.html`, `.nojekyll`, and `README.md`, then **Commit changes**.
   (Or, if you have `git` installed: `git init`, `git add -A`, `git commit -m "sleep tracker"`,
   `git remote add origin https://github.com/<you>/<repo>.git`, `git push -u origin main`.)
3. Go to the repo's **Settings** tab → **Pages** in the left sidebar.
4. Under **Build and deployment → Source**, choose **Deploy from a branch**.
5. Under **Branch**, choose `main` and folder `/ (root)`, then **Save**.
6. Wait about a minute, then refresh that Settings → Pages screen — it'll show
   *"Your site is live at"* followed by a URL:
   `https://<you>.github.io/<repo>/`. That's your app.

Write that URL down — you'll paste it into Google Cloud Console in the next section.
Add the page to your phone's home screen and the log button is one tap from the lock screen.

## Importing Apple Watch deep sleep

On your iPhone: **Health → profile picture → Export All Health Data**. That produces
`export.zip`; unzip it and drop `apple_health_export/export.xml` onto the import box.

The file is often hundreds of megabytes, so it's read in 8 MB chunks rather than
loaded whole. Parsing happens entirely in your browser — nothing is uploaded.

- Only `HKCategoryTypeIdentifierSleepAnalysis` records are read.
- Deep sleep is summed from `AsleepDeep` segments per night. When two devices
  recorded the same night, the one reporting the most is used rather than summing
  them, which would double-count.
- Nights are assigned using the wall-clock time where the record was made, so
  travel across timezones doesn't shift them.
- *Backfill missing bedtimes* (checkbox) fills in the earliest in-bed/asleep time
  for nights you never logged by hand. It never overwrites a bedtime you logged
  yourself.

**CSV** also works, for apps like Health Auto Export or Bevel. It needs a date
column and a deep-sleep column; values may be `83`, `1.4` (with an hours-flavoured
header), `1h 23m`, or `1:23`.

## Where your data lives

In this browser's `localStorage`, under the key `sleep-tracker-v1` — **per-browser
and per-device** by default, and clearing site data will erase it. Two ways to carry
it across devices:

- **Manual**: **Export JSON** for a backup, drop the file in Google Drive (or any
  synced folder), **Import JSON** on another device. Import merges by date, so
  re-importing an older backup won't delete newer nights.
- **Automatic**: connect Google Drive (below) and it saves itself, in the
  background, every time you log something.

## Google Drive sync

This needs a one-time setup in Google Cloud Console, because Google requires every
app that touches Drive to register itself and identify who's allowed to sign into
it. It's five minutes, and it's only ever done once. **Do the GitHub Pages section
above first** — you need that live URL for step 3 below.

**1. Create a Google Cloud project**

1. Go to [console.cloud.google.com](https://console.cloud.google.com/) and sign in
   with the Google account whose Drive you want to sync to.
2. Top-left, next to "Google Cloud", click the project dropdown → **New Project**.
3. Name it anything (e.g. `sleep-tracker`) → **Create**. Wait a few seconds, then
   make sure the dropdown at the top shows this new project selected.

**2. Enable the Drive API**

1. In the left sidebar (or the ☰ menu), go to **APIs & Services → Library**.
2. Search for **Google Drive API** → click it → **Enable**.

**3. Configure the OAuth consent screen**

1. **APIs & Services → OAuth consent screen**.
2. User type: **External** → **Create**.
3. Fill in the required fields: **App name** (e.g. `Sleep Tracker`), **User support
   email** (your email), and **Developer contact email** (your email again) →
   **Save and Continue**.
4. On the **Scopes** step, click **Add or Remove Scopes**, search for `drive.file`,
   check **`.../auth/drive.file`** (*"See, edit, create, and delete only the
   specific Google Drive files you use with this app"* — the narrow, non-sensitive
   scope; it can only ever touch the one file this app creates, nothing else in
   your Drive) → **Update** → **Save and Continue**.
5. On the **Test users** step, click **Add Users** and add your own Google account's
   email address → **Save and Continue**.
6. Leave **Publishing status** as **Testing**. You do not need to submit this for
   Google's verification — that's only required for apps used by the public.
   Testing mode works indefinitely for the account(s) you listed as test users.

**4. Create the OAuth Client ID**

1. **APIs & Services → Credentials → Create Credentials → OAuth client ID**.
2. Application type: **Web application**. Name it anything.
3. Under **Authorized JavaScript origins**, click **Add URI** and paste your GitHub
   Pages origin **with no trailing slash and no path** —
   e.g. `https://yourname.github.io` (not `.../sleep-tracker/`).
4. Click **Create**. A dialog shows **Your Client ID** — copy it
   (looks like `123456789-abc...apps.googleusercontent.com`). You don't need the
   client secret; this app never uses it.

**5. Paste the Client ID into the app**

1. Open `index.html`, find this line near the top of the `<script>` block:
   ```js
   const GOOGLE_CLIENT_ID = "";
   ```
2. Paste your Client ID between the quotes, save, and re-upload/push `index.html`
   to your GitHub repo (Add file → Upload files, or `git add -A && git commit -m
   "add drive client id" && git push`).
3. Give GitHub Pages a minute to redeploy, then reload your app.

**6. Connect it**

In the app's **Your data** card, click **Connect Google Drive**. The first time,
Google shows a warning that the app is unverified — that's expected for a personal
app in Testing mode; click **Advanced → Go to \<your app name\> (unsafe)** (whatever
you named it in step 1) — "unsafe" here just means Google hasn't manually reviewed
it, not that anything is actually wrong. Approve the `drive.file` permission. It
creates `sleep-tracker-data.json` in your My Drive — one file for sleep, weight,
and exercise together — and syncs to it from then on; open the
same URL on another device (signed into the same Google account) and click Connect
there too — it downloads that file and merges it with whatever's already local.

**How the sync works, and its limits:** it's one JSON file, updated a couple of
seconds after any change, and each night's or day's entry merges field-by-field
rather than overwriting wholesale. It is *not* real-time — two devices editing the
exact same night or day within the same few seconds can have one edit win over the
other. For one
person's own devices this is more than good enough; it is not built for multiple
people editing concurrently.

**Revoking access later:** [myaccount.google.com/permissions](https://myaccount.google.com/permissions)
→ find the app → **Remove Access**. The **Disconnect** button in the app just ends
the current browser session; it doesn't touch the Drive file or the permission grant.

## Data format

```json
{
  "version": 2,
  "settings": {
    "targetBedtime": "22:30", "minDeepMinutes": 90,
    "liftLabelA": "Day 1", "liftLabelB": "Day 2"
  },
  "nights": {
    "2026-08-02": { "bedtime": "23:09", "deepMinutes": 117, "source": "apple" }
  },
  "days": {
    "2026-08-02": {
      "weightKg": 72.35,
      "lift": "a",
      "cardioMinutes": 30,
      "activities": [{ "name": "Tennis", "minutes": 60 }]
    }
  }
}
```

`nights` is keyed by the *night of* date (a bedtime after midnight files under the
previous night); `days` is keyed by the calendar date you logged weight/exercise
against. `lift` is `"a"` or `"b"`, matching whichever of `liftLabelA` / `liftLabelB`
you did that day; omit the field (or leave it out entirely) for a rest day.
Everything is plain and hand-editable.

## Notes on the charts

Bedtime is plotted on an axis anchored at 18:00, so times either side of midnight
stay in order instead of jumping from 23:59 to 00:00. Every chart section — Weight,
Activity, Bedtime, Deep sleep — is its own single-series plot on its own axis
rather than one shared multi-axis chart; overlaying series with different units on
one plot would visually imply a correlation the data doesn't actually contain.
What they *do* share is the date range and per-day horizontal alignment, so you
can still read across them — see "Looking for correlation" above.

The chart colors are checked against the colorblind-separation, lightness,
chroma, and contrast gates:

```
python tools/validate_palette.py "#2a78d6,#1baf7a,#e0a30e" --mode light --pairs all
python tools/validate_palette.py "#3987e5,#199e70,#bb7e12" --mode dark  --pairs all
```

Both pass in both themes (bed = blue, deep sleep = green, weight = amber). The
Activity strip deliberately doesn't add more hues on top of that: the lift-day
squares use plain ink with a "1"/"2" digit inside (not color) to tell the two days
apart, and the cardio/other-activity bars reuse the weight amber at two different
opacities, distinguished by their row label rather than a new color. Re-run the
validator if you change any of the three chart colors — several plausible-looking
combinations (blue + violet, for one) collapse to near-identical under red-green
colorblindness, especially in dark mode.

`tools/make_preview.py` builds a copy of the app seeded with 45 nights of sample
data, for eyeballing layout changes without touching your real history:

```
python tools/make_preview.py light 30     # → _preview_light_30.html
```

Preview files are throwaway; delete them when you're done.
