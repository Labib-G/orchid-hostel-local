# Orchid Hostel — Setup Guide

A free Android "app" for running Orchid Hostel: seats, residents, biodata & attachments, monthly
dues, and a Gemini-powered voice assistant that can do most of the above by voice or text.

It's built as a **PWA** (Progressive Web App) rather than a Play-Store app. That's the realistic
$0 route — a native app needs Android Studio to compile and a $25 one-time Play Console fee to
publish. A PWA installs on the phone with its own icon, opens full-screen with no browser bar,
and works offline — but costs nothing and needs no developer account.

---

## 1. Try it immediately (2 minutes, no setup)

1. Open `index.html` (double-click it, or open it in Chrome).
2. It works right away in **Local mode** — data is saved on that one device/browser only,
   using its storage. Add a room, add a resident, play with it.
3. Nothing is connected to the internet yet except Google Fonts and (later) the voice
   assistant — your data stays on the device.

This is enough to evaluate the app. For real daily use you'll want to put it online (Step 2)
so you can install it on your phone properly and, optionally, sync data across devices (Step 4).

---

## 2. Put it online for free (so it can install as a real app)

Android's "install as app" feature needs the app served over `https://`, not opened as a local
file. **GitHub Pages** is the easiest free option — no card, no command line.

1. Create a free account at [github.com](https://github.com) if you don't have one.
2. Click **+ → New repository**. Name it e.g. `orchid-hostel`. Keep it Public. Create it.
3. Click **Add file → Upload files**, then drag in all 5 files from this folder:
   `index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`. Commit.
4. Go to **Settings → Pages**. Under "Build and deployment", set **Source: Deploy from a
   branch**, branch **main**, folder **/ (root)**. Save.
5. Wait about a minute, then open the URL shown there — something like
   `https://YOUR-USERNAME.github.io/orchid-hostel/`.

That link is now your hostel's app, live and free, forever (GitHub Pages has no usage limit
that a hostel would ever hit).

### Install it on Android
Open that link in **Chrome** on the phone → tap the **⋮** menu → **"Add to Home screen" / "Install
app"**. It now behaves like any other app: its own icon, opens full-screen, works offline for
viewing data already loaded.

---

## 3. Give it a free Gemini API key (for the voice assistant)

1. Go to **[aistudio.google.com](https://aistudio.google.com)**, sign in with any Google account.
2. Click **"Get API key" → "Create API key"**. Copy it (starts with `AIza...`).
3. In the app, go to **Settings → Gemini voice assistant**, paste the key, tap **Save key**.

The free tier is generous for a single hostel's daily use. The key is stored only on that
device (in the browser's local storage) — it is never sent anywhere except directly to Google
when you use the mic.

Tap the **mic button** and try things like:
- *"How many seats are empty?"*
- *"Add a new resident named Rahim, phone 017xxxxxxx, room 101 seat 2, rent 5000."*
- *"Mark Rahim's payment as paid for August."*
- *"Who still has dues this month?"*
- *"Vacate room 102 seat 1."*

It also understands Bengali if you switch **Settings → Voice command language** to বাংলা.
No mic? There's a text box in the same voice window — type the command instead.

*(If Google ever retires the `gemini-3.7-flash` model name, open `index.html`, search for
`GEMINI_MODEL`, and swap in whatever the current fast Gemini model is listed at
[ai.google.dev/gemini-api/docs/models](https://ai.google.dev/gemini-api/docs/models).)*

---

## 4. Optional: sync data across devices with Firebase (still $0)

By default all data lives in that one browser's local storage. If you want the data to survive
a reinstall, or want two phones (e.g. you and a staff member) seeing the same live data, connect
a free Firebase project. **This uses only Firestore (database) and Authentication (login) — not
Firebase Storage**, because Google now requires a billing card for Storage even on the free
tier. Firestore + Auth need **no card at all**.

For this build the **Firebase config is already embedded in `index.html`** (`FB_DEFAULT_CONFIG`),
pointing at the shared `orchid-hostel` project that backs your customer site
(`orchid-hostel.web.app`). So steps 1–6 below are already done for you — you only need to enable
Auth/email-password and set the Firestore rules.

1. Go to **[console.firebase.google.com](https://console.firebase.google.com)**, open the
   `orchid-hostel` project.
2. **Build → Authentication → Sign-in method → Email/Password → Enable**.
3. **Build → Firestore Database** → if not present, create it (production mode, a region near
   your hostel).
4. **Build → Firestore Database → Rules** tab — paste the rules from step 5 below, then **Publish**.
5. Firestore rules. Start with the open default so sign-in works, then harden to the admin UID
   once the admin account exists (see step 7):
   ```js
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
   (only someone signed in to your app can read or write data.)
6. The config is embedded, so there's nothing to paste in the app — just open it.
7. The app will ask you to sign in — tap **Create account**, set an email + password for
   yourself (this is the one admin login for your hostel — don't share the sign-up screen
   publicly). From then on, sign in with that email/password on any device to see the same data.
   After the account is created, get its UID from **Authentication → Users** and lock the rules down
   so only that admin can edit rooms/residents while customers still read them:
   ```js
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read: if true;                                  // customers see rooms
         allow write: if request.auth.uid == 'PASTE_ADMIN_UID_HERE';
       }
     }
   }
   ```
   The admin app reads/writes the collections `residents`, `seats`, `payments`, `meta/settings`
   (`index.html` `DB` object) — make sure your customer site reads the same names.

> **Admin-only note:** this PWA is the *staff/admin* tool — it has no customer-facing checkout,
> payment, or booking UI. Guest booking is handled by your separate `orchid-hostel.web.app` site,
> which reads the **same** Firestore data (rooms, availability, rent) via this shared project.

Firestore's free quota (50K reads / 20K writes per day, 1GB storage) is far more than a single
hostel will ever use, so this genuinely stays $0.

---

## What's included

| File | Purpose |
|---|---|
| `index.html` | The whole app (UI + logic). This is the only file you ever need to edit. |
| `manifest.json` | Tells Android how to install it (name, icon, colors). |
| `sw.js` | Service worker — caches the app shell so it opens instantly and works offline. |
| `icon-192.png` / `icon-512.png` | App icon. |

## Features

- **Dashboard** — total due, resident count, seats empty/occupied, and a "money received this
  cycle" ring showing the actual amount collected vs. expected for the month.
- **Seats** — rooms shown as key-tag tiles, always listing empty seats before occupied ones
  within a room, freshly reset every time you open the app. A search bar finds a seat by room
  number, resident name, or phone. Sort chips reorder rooms by emptiest-first, occupied-first,
  floor, or bed type (set floor/bed type when adding a room). Tap a tile to assign, view, or
  vacate a seat — a red dot marks an occupied seat whose resident owes money this month.
- **Residents** — add/edit, search & filter (all / active / due / paid / left — search also
  matches a room number), full biodata (phone, NID, guardian, address, monthly rent, join date),
  attachments (PDF/PNG/JPG — uploadable right from the Add Resident form), room/seat assignment
  built into the same form, and full payment history per resident.
- **Trash** — deleting a resident frees their seat immediately but keeps their data for 40 days
  (Settings → Trash, or the Trash link on the Residents page) with one-tap Restore, before it's
  permanently removed. Nothing is erased on the spot.
- **Dues** — pick any month, one tap to generate that month's due entries for every active
  resident from their rent, see collected vs outstanding, tap any resident to record a payment.
- **Voice assistant** — the mic button (or the text box inside it) understands natural requests
  in English or Bengali and can look up, add, or update almost anything in the app — including
  deleting (to Trash) and creating rooms.
- **Sync** — Settings → Cloud sync connects this admin app to a Firebase project (Firestore
  Auth), so rooms/residents/payments live in one database. Point it at your existing
  `orchid-hostel.web.app` project and the customer site and admin app share data automatically.
- **Dark mode by default** — a moon/sun toggle in the top bar (and a Light/Dark/System picker in
  Settings) if you'd rather switch it.

### A note on attachments
There's no separate file-storage service (again, to avoid needing a billing card), so uploaded
files are kept as small in-database blobs. Keep them modest — a compressed photo of an ID card,
not a full-resolution scan. Local mode allows up to ~4MB per file; once cloud sync is on, it's
capped at 700KB per file (Firestore's per-document limit is 1MB total).

### Costs, honestly
- GitHub Pages hosting: free, no limits you'd hit.
- Firebase Firestore + Auth (Spark plan): free, no card required, generous daily quotas.
- Gemini API: free tier, generous for one hostel's daily voice commands.
- Total: **$0**, no card on file anywhere in this stack.

If you outgrow the free quotas (very unlikely for a single hostel), each service tells you
before charging anything — none of them auto-upgrade to a paid plan.
