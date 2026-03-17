# Meltify

**Your personal radio station. Curated. No algorithm.**

A working example is live at [meltify.org](https://meltify.org) — open it, listen, and tap BYO in the footer to get the full build prompt and companion guide.

This repo is the template. Fork it, build your own station, put in only artists you'd stand behind.

---

## Quick start

1. Read the companion guide below
2. Copy the build prompt from [meltify.org/byo](https://meltify.org) 
3. Paste it into Claude, ChatGPT or Gemini
4. Follow the conversation — it walks you through everything

The only cost is a domain name — about €5. Everything else is free.

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire player — all panels, all logic, one file |
| `admin.html` | Password-protected admin panel for adding music from Archive.org |
| `config.js` | Your credentials — Supabase URL, anon key, admin password |
| `manifest.json` | Makes it installable as a PWA on any device |
| `favicon.svg` | The logo |

---

## Companion guide

> This guide walks you through building your own curated radio station from zero. Keep it open beside your AI conversation as you build. The AI does the building — this document helps you understand what is happening at each stage, what to expect, and what to do if something goes wrong.

### Before you start

You need four accounts. All are free. Sign up for all four before you begin.

**Supabase** — supabase.com — your database. Sign in with GitHub. The browser talks directly to Supabase using fetch calls over HTTPS — no backend server needed.

**GitHub** — github.com — where your code lives. Every push redeploys your site automatically within seconds.

**Cloudflare** — cloudflare.com — where your site is hosted. A Cloudflare Worker serves your single HTML file as a live website. Free tier is more than enough.

**Namecheap** — namecheap.com — where you buy your domain name. Approximately €5 per year. After buying, point the nameservers to Cloudflare.

---

### Stage 1 — Set up Supabase

**What you are doing:** creating a free database with four tables that the player will read from and write to.

**What to expect:** the AI will ask you to create a project at supabase.com, then give you SQL to run in the Supabase SQL editor. Copy and paste the SQL exactly as given and click Run.

**The four tables:**

- `tracks` — every song in your radio station
- `melts` — a log of every track any listener has melted, stored against their anonymous MELT code
- `molts` — a log of every track molted (removed permanently from a device's shuffle)
- `user_curations` — each listener's personal collection of melted tracks, queryable by their MELT code

**What you need to copy and save:** after creating the project, go to Settings → Data API. Copy and save your Project URL and anon/public key. You will need both in every step.

**Disabling Row Level Security:** go to Authentication → Policies and disable Row Level Security on all four tables. Safe here because no personal data is stored — only track URLs, anonymous codes, and artist names.

**Done when:** all four tables exist, RLS is disabled on all four, and you have your Project URL and anon key saved.

---

### Stage 2 — Set up GitHub and Cloudflare Workers

**What you are doing:** creating a home for your code and a live URL for your station.

**GitHub:** create a new empty repository. The AI will tell you exactly what files to create.

**config.js:** contains your Supabase URL, anon key, and an admin password. The anon key is safe to include — it is designed to be public and only allows the operations you have set up. Never use your Supabase service role key in client-side code.

**Cloudflare Workers:** go to Workers & Pages → Create → Worker. The AI gives you a short worker script to paste in. Store your `index.html` and `config.js` contents as environment variables named `INDEX_HTML` and `CONFIG_JS`.

**Connecting your domain:** add your domain in Cloudflare, point Namecheap's nameservers to Cloudflare, then connect the domain to your Worker. Usually live within minutes.

**Done when:** your domain loads in a browser.

---

### Stage 3 — Build the player

**What you are doing:** building the main HTML file — the player, all panels, all logic, in one file.

The AI will write a large single HTML file. This is normal. No frameworks, no build step.

Design system: pure black `#000000`, JetBrains Mono font, green accent `#1DB954`. Artwork fills the top of the screen with track info overlaid. Below: progress bar, controls (prev · play · skip), melt and molt buttons, footer with links.

All panels — info overlay, curations, BYO, privacy — are full-page replacements. The footer disappears when a panel is open. Nothing overlaps.

**Testing locally:** never open index.html by double-clicking — `file://` blocks fetch calls. Always test through a local server:
```
npx serve .
```
Then open `http://localhost:3000`.

**Done when:** your domain shows the player — black screen, artwork area, controls, footer.

---

### Stage 4 — Add music through the admin panel

**What you are doing:** building admin.html — a password-protected page for searching Archive.org and adding tracks to your database.

Archive.org has millions of freely and legally streamable music files. The API is completely open — no API key, no signup, no cost.

**How it works:**

1. Go to `yourdomain.com/admin.html` and enter your admin password
2. Search for an artist — the panel shows matching albums with artwork and track counts
3. Tick albums, click Batch Add — it fetches MP3 files and inserts them into Supabase
4. Or paste any `archive.org/details/IDENTIFIER` URL directly to add a specific album
5. Use Manage Tracks to filter, review and remove anything you don't want

**Finding music:** try searching krautrock, drone, free jazz, experimental, psych folk, motorik. The netlabels collection is particularly good for underground material under Creative Commons.

**Done when:** you have at least 20–30 tracks in Supabase and the player loads and shuffles them.

---

### Stage 5 — Melt and Molt

**What you are doing:** verifying the personal curation system works end to end.

**Melt** adds the current track to the listener's personal curation. First tap shows an explanation sheet. After that, fires immediately with a green confirmation line. Saves locally and to Supabase against the listener's anonymous MELT code. Duplicate check prevents the same track being added twice.

**Molt** removes the current track permanently from this device's shuffle. Never plays again on this device. Logs to Supabase but never syncs between devices.

**Testing:**
1. Play a track. Tap Melt. See the explanation sheet, then "melted into your curation ✓"
2. Tap Melt again on the same track — see "already in your curation"
3. Tap Molt — track disappears from shuffle
4. Open Curations — melted track appears with your MELT code below
5. Check Supabase melts and user_curations tables — rows should be there

**Done when:** melt and molt work, duplicates are blocked, data appears in Supabase.

---

### Stage 6 — Curations panel

**What you are doing:** verifying the full personal curation experience.

The panel shows: your melted tracks with remove buttons, a Play My Curation button that switches the player to curation-only mode, your MELT code, and a transfer section for loading a curation on a new device.

**Testing transfer between devices:**
1. Melt several tracks on your desktop
2. Note your MELT code from the Curations panel
3. Open your station on your phone
4. Go to Curations, tap "use on another device ↓", enter your code, tap Load
5. Your melted tracks appear on your phone

**Done when:** Play My Curation works, transfer between devices works, removing tracks works.

---

### Stage 7 — Info overlay and BYO panel

**What you are doing:** personalising the station with your own writing, and confirming the BYO prompt copies correctly.

**Info overlay:** tap the ⓘ icon on the artwork. Edit the station description, current curation details, and About the curator section to describe your own station and taste.

**BYO panel:** tap BYO in the footer. Confirm the Copy prompt button works, the companion guide downloads, and the GitHub link opens correctly.

**Done when:** the info overlay has your own writing and both buttons work.

---

### Stage 8 — PWA (add to home screen)

**What you are doing:** making the station installable as an app on any device.

**Android (Chrome):** three-dot menu → Add to home screen

**iPhone (Safari):** share icon → Add to Home Screen

**Desktop (Chrome/Edge):** install icon in the address bar

**Done when:** your station is installed on your phone as an app icon and opens fullscreen.

---

### Common problems and fixes

**Player loads but no tracks:** check your Supabase tracks table has rows. If empty, add music through the admin panel first.

**Tracks skip constantly:** Archive.org stream failures. Normal for some items. The auto-skip handles it. If it stops after 4 skips, your connection may be struggling — try again later.

**Melt doesn't save to Supabase:** open browser console (F12). Red errors usually mean your Supabase URL or anon key in config.js is wrong or has extra spaces.

**Admin panel finds no tracks:** Archive.org creator searches are case-sensitive. Try different spellings, or use the URL paste box with a direct `archive.org/details` link.

**Changes not showing after deploy:** update the `INDEX_HTML` environment variable in your Cloudflare Worker and redeploy. Hard reload (Ctrl+Shift+R).

**Transfer input not visible on mobile:** tap "use on another device ↓" — collapsed by default on mobile.

---

### What it costs

| Item | Cost |
|------|------|
| Domain name (Namecheap) | ~€5/year |
| Supabase | €0 |
| Cloudflare Workers | €0 |
| GitHub | €0 |
| Archive.org music | €0 |
| **Total** | **~€5/year** |

---

### Where to go next

**As a listener:** keep melting and molting. Your curation gets better over time. Share the URL with people whose taste you respect.

**As a builder:** build themed exhibitions through the admin panel — a month of motorik, a week of home-recorded folk, short experimental pieces. Each exhibition is a fresh start for your listeners.

**As a developer:** the peer.school canvas has scraping instruments for more sophisticated discovery — by duration, year, subject tag, archive collection. Contact ross@peer.school for access.

---

## Ideas and extensions

This is a single HTML file with a Supabase backend. The architecture is intentionally simple so it can go anywhere. Some directions people might take it:

**Offline mode** — a service worker could cache the playlist and recent tracks so the station works without a connection. PWA offline mode is well-supported and would make this feel genuinely native.

**Native apps** — the PWA is good enough for most use cases, but the codebase could be wrapped with Capacitor or Tauri to produce a proper iOS, Android or desktop app. Lock screen controls, background audio, system media integration all become available.

**Kodi plugin** — the Supabase tracks table is a clean JSON API. A Kodi plugin that reads from it directly would make any Meltify station playable through a home media centre with no extra work.

**Library of curations** — the current architecture supports one active curation at a time. A small extension to the Supabase schema could support multiple named curations per station — genre collections, mood playlists, themed exhibitions — switchable from the Curations panel.

**Public curation sharing** — MELT codes are already anonymous identifiers. A discovery page that lists publicly shared curations by code would let the community find each other's stations without any login system.

**Collaborative curations** — multiple curators contributing melts to a shared MELT code. The schema already supports it — user_curations is keyed by user_id, which could be a shared group code rather than an individual one.

**Scrobbling** — the play events are already tracked locally. Posting to Last.fm or ListenBrainz on each track play would take about 20 lines of code.

---

## Philosophy

Put in only artists you'd stand behind. Keep it small and honest. Share it with people who'll appreciate it. Don't monetise it.

The whole thing costs nothing to run and about a day to build.

---

## Licence

MIT — do whatever you want with it.

---

*Built by [Ross Rabette](https://peer.school) — an engineer with 30 years experience and a strong opinion about music.*
