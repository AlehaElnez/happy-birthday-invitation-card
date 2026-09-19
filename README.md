# Birthday Invitation

A one-page invitation that opens with a wax-sealed envelope, counts down to the
day, and lets guests register themselves. Every registration is saved to a
database and appears in the guest list for everyone to see.

```
index.html     the entire invitation — design, wording, countdown, envelope, form
api/rsvps.js   the backend that saves and returns registrations
README.md      this file
```

There are no image files. The envelope, the wax seal and the laurel are drawn in
SVG inside `index.html`, so nothing can go missing and nothing needs uploading.

---

# Part 1 · Getting it online

### Step 1 — Put the files on GitHub

1. Go to <https://github.com/new>, name it (say `birthday-invite`), leave it
   empty (no README, no .gitignore), and create it.
2. On the empty repo page click **uploading an existing file**, drag in
   `index.html`, `README.md` and the whole `api` folder, then **Commit changes**.

Prefer the terminal?

```bash
cd path/to/this/folder
git init
git add .
git commit -m "Birthday invitation"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/birthday-invite.git
git push -u origin main
```

### Step 2 — Deploy on Vercel

1. Sign in at <https://vercel.com> with GitHub.
2. **Add New → Project**, choose the repo, click **Deploy**.
3. Change nothing in the build settings. There is no framework and no build
   step — Vercel serves `index.html` and turns `api/rsvps.js` into a serverless
   function on its own.

Your invitation is live. The form won't save yet — that's Step 3.

### Step 3 — Connect the database

This is the step that makes registration work.

1. Open your project in Vercel → the **Storage** tab.
2. **Create Database → Upstash for Redis** (in the Marketplace, free tier is
   plenty). Name it, pick the region closest to your guests, create it.
3. When prompted, **connect it to this project** and tick all three
   environments (Production, Preview, Development).

Connecting it creates two environment variables for you:
`KV_REST_API_URL` and `KV_REST_API_TOKEN`. The code looks for exactly those
names, so there is nothing to copy or paste.

### Step 4 — Set your host key

1. **Settings → Environment Variables**.
2. Name: `HOST_KEY` — Value: any password you pick. Tick all environments. Save.

That key opens your private dashboard at `your-site.vercel.app/#admin`, where
you see every response with phone numbers and can download a CSV. Guests never
see contact details; the public guest list shows only name, seat count and note.

### Step 5 — Redeploy once

**Deployments → the newest one → ⋯ → Redeploy.**
Environment variables only reach the function on a fresh deployment. Skipping
this is the single most common reason the form still shows an error.

### Step 6 — Test it

- Open the site, tap the envelope, scroll to RSVP, register yourself.
- Your name should appear under "The guest list" straight away.
- Visit `/#admin`, enter your `HOST_KEY`, confirm the row is there.
- Delete that test entry later if you like — see "Clearing test entries" below.

### Step 7 — Your own domain (optional)

**Settings → Domains**, add a domain you own, and follow the DNS instructions
Vercel shows you.

---

# Part 2 · Changing things to suit you

Everything you'd normally want to change lives in two clearly marked blocks
inside `index.html`. You can edit the file on GitHub directly (pencil icon →
commit) and Vercel redeploys in under a minute.

### Name, date, time, place and wording

Near the bottom of the file, look for:

```
/* 2 · PARTY SETTINGS — CHANGE THE NAME, DATE, TIME AND PLACE HERE */
const PARTY = { ... }
```

| Setting | What it does |
|---|---|
| `name` | The big name on the card and the hero |
| `occasion` | Small line under the name, e.g. "Turns Thirty" |
| `monogram` | The letter printed on the envelope flap |
| `startsAt` / `endsAt` | **Drive the countdown and the calendar file.** Real date/time |
| `dateText`, `timeText`, `whenLine` | The date and time as written out for readers |
| `countdownOver` | What the timer says once the day arrives |
| `venueName`, `venueAddress`, `venueLine` | The place |
| `mapsQuery` | What the "Open in Google Maps" button searches for |
| `coverSubtitle`, `openLabel` | The two captions under the envelope |
| `heroKicker`, `noteKicker`, `noteHeading`, `noteBody` | The invitation wording |
| `rsvpDeadline` | Line above the RSVP form |
| `footerSign`, `footerLine` | The sign-off |
| `showEnvelopeOnce` | `true` skips the envelope if the guest already opened it this visit; set `false` to show it every time |

**About the date format.** `startsAt` must look like
`"2026-10-24T19:00:00+05:30"` — year-month-day, then `T`, then 24-hour time,
then your timezone offset:

`+05:30` Sri Lanka & India · `+04:00` Dubai · `+01:00` Berlin · `+00:00` London
`-05:00` New York · `-08:00` Los Angeles · `+08:00` Singapore · `+10:00` Sydney

Use `\n` inside a line to break it onto two lines (as `venueAddress` does).

### Colours

At the very top of the `<style>` section:

```
/* 1 · COLOUR SETTINGS — CHANGE ANY COLOUR HERE */
:root{ ... }
```

Every colour is listed once with a comment saying what it controls, grouped
into: page backgrounds, text, gold accent, lines/cards/fields, the envelope
screen, and the wax seal. Change a hex value and it updates everywhere it is
used, including the SVG envelope.

A few you'll likely reach for first:

- `--accent` and `--accent-light` — the gold on buttons, rules and the timer
- `--bg-main` / `--bg-alt` — the two alternating section backgrounds
- `--wax-mid` — the main wax colour (try `#8d2f32` for classic red wax, and set
  `--wax-light` `#b8555a`, `--wax-dark` `#5c1a1d` to match)
- `--env-screen` — the background behind the envelope

### The envelope itself

- **The letter on the flap** is `monogram` in the party settings.
- **How big it is:** find `.envelope{` in the CSS. `width:min(82vw,46svh,400px)`
  means "84% of the screen width, or 46% of the screen height, or 400px —
  whichever is smallest". Raise the last number to make it bigger on desktop.
- **How fast it opens:** the animation lines just below —
  `sealBreak`, `flapOpen`, `cardRise`, `pullBack`. The numbers are seconds
  (duration first, then delay).
- **The card inside** is the block commented "the invitation card"; it shows the
  name, occasion and date automatically.

---

# Part 3 · Good to know

**Testing locally.** Double-clicking `index.html` opens it in your browser and
the envelope, countdown and layout all work — it will say "design preview"
where the guest list goes, because registrations need the deployed backend. To
run the backend on your own machine: `npm i -g vercel`, then `vercel dev` in
this folder.

**Guest privacy.** `/api/rsvps` returns only names, seat counts and notes to the
public page. Contact details are returned only when the correct `HOST_KEY` is
supplied, so they are never exposed in the browser.

**No guest logins.** Anyone with the link can register, and nothing prevents the
same person registering twice.

**Removing a registration.** Only you can do this, from the dashboard. Go to
`your-site.vercel.app/#admin`, enter your `HOST_KEY`, and each row in the table
has a **Remove** button on the right. It asks you to confirm, then deletes that
entry from the database — it disappears from the dashboard and, if they'd said
yes, from the public guest list too, immediately. This can't be undone, so it's
worth double-checking the name before confirming. Guests never see this button;
it only appears once you're logged into the dashboard with the correct key.

**Clearing test entries.** In Upstash, open your database → **Data Browser** →
delete the `rsvps` key to empty the list completely.

**Capacity.** The most recent 1000 responses are kept.

**Updating later.** Edit on GitHub and commit; Vercel redeploys automatically.

---

# If something goes wrong

| What you see | What it means |
|---|---|
| "That didn't save" on the form | The database isn't connected, or you didn't redeploy after connecting it. Redo Steps 3 and 5. |
| "The guest list isn't loading" | Same cause. Open **Logs** in Vercel to see the exact message. |
| "Storage is not configured" in the logs | `KV_REST_API_URL` / `KV_REST_API_TOKEN` are missing — the Upstash database isn't linked to this project. |
| "That key doesn't match" | `HOST_KEY` isn't set in Vercel, or you set it after the last deployment. |
| Envelope doesn't animate | The guest has "reduce motion" enabled on their device — it skips straight to the invitation on purpose. |
| Fonts look plain | The device couldn't reach Google Fonts; the layout falls back to a system serif and still works. |
