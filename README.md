# DentCase — deployment guide

This folder is a complete, installable web app:
- `index.html` — the app itself
- `manifest.json` — makes it installable to a phone home screen
- `sw.js` — offline support
- `icons/` — app icons

Storage now works **standalone** (no dependency on Claude): it uses your
browser's local storage by default, with optional free Supabase cloud sync
so data survives across devices and can be shared with a team.

---

## Step 1 — Host it (pick one, both are free)

**Option A: Netlify Drop (fastest, no signup)**
1. Go to https://app.netlify.com/drop
2. Drag this entire folder onto the page
3. You'll get a live URL like `https://random-name.netlify.app` in seconds

**Option B: Replit**
1. Go to replit.com → Create → "Static Site" (or "HTML/CSS/JS")
2. Upload all files in this folder, keeping the folder structure
   (`icons/` must stay as a subfolder)
3. Click Run — Replit gives you a live URL like `https://dentcase.yourname.repl.co`

Either way, open the URL on your phone and you should see the app.
On Android Chrome, you'll get an "Install app" prompt — that's the PWA
manifest working.

---

## Step 2 — Set up free cloud storage (Supabase)

Skip this if you only need it on one device — localStorage alone works fine
for solo use. Do this if you want data to survive a browser reset, sync
across your phone/laptop, or share a workspace with colleagues.

1. Go to https://supabase.com → sign up free → "New project"
2. Once created, go to the **SQL Editor** and run this once:

```sql
create table kv_store (
  key text primary key,
  value text,
  updated_at timestamptz default now()
);

alter table kv_store enable row level security;

create policy "Allow anon read/write"
  on kv_store for all
  using (true)
  with check (true);
```

   ⚠️ This policy allows anyone with your anon key to read/write this table.
   That's fine for a small trusted team (same tradeoff as the workspace code
   feature), but don't put this in front of a large public deployment without
   tightening the policy.

3. Go to **Project Settings → API**. Copy:
   - **Project URL** (looks like `https://xxxxx.supabase.co`)
   - **anon public key** (a long string starting with `eyJ...`)
4. Open DentCase → tap **⚙ → Cloud sync** → paste both in → Save.

From then on, every save also syncs to Supabase. If Supabase is ever
unreachable, the app quietly falls back to the local copy so you never lose
data mid-consultation.

---

## Step 3 — Wrap it as an Android app (APK/AAB) for Play Store

Once your app has a live HTTPS URL (from Step 1):

1. Go to https://www.pwabuilder.com
2. Paste your URL, click "Start"
3. PWABuilder scores your PWA (manifest + service worker are already done
   for you) and generates a signed **Android package (AAB)** — click
   "Package for Stores" → Android
4. Download the package

## Step 4 — Publish to Play Store

1. Create a Google Play Developer account (one-time $25):
   https://play.google.com/console/signup
2. Create a new app, fill in the store listing (screenshots, description,
   privacy policy — required even for a simple app)
3. Upload the AAB from Step 3 under "Production" (or start with "Internal
   testing" to try it privately first — recommended)
4. Submit for review

A privacy policy is mandatory since this app handles patient data — even a
simple one-page policy stating what's collected (name, age, case notes,
photos) and that it's stored via Supabase satisfies the requirement. Happy
to draft that with you when you're ready for this step.

---

## Honest limitations to know about

- The **workspace code** and **Cloud sync** together give you shared data
  across a small trusted team, but there's no real login/password — anyone
  with the code or key can read/write everything. Fine for a resident team
  sharing one supervised unit; not a substitute for a properly access-controlled
  hospital EMR.
- Voice dictation depends on the phone's browser supporting the Web Speech
  API — works on Chrome/Edge, may not work on all browsers.
- Images are compressed before storing to keep sync fast — original photo
  quality/resolution is not preserved.
