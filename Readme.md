# The Guitar Room — Web App & PWA

**A complete digital presence for The Guitar Room, Chennai** — a marketing website, an installable practice app, and a live student-review pipeline, built as a single self-contained, zero-dependency-cost product.

🔗 **Live site:** `https://dhevdharsan.github.io/guitar-room-website/` *(GitHub Pages)*
🎸 **Repository:** `guitar-room-website`

---

## Table of Contents

1. [Overview](#overview)
2. [Feature Tour](#feature-tour)
3. [Tech Stack](#tech-stack)
4. [Project Structure](#project-structure)
5. [Architecture](#architecture)
6. [Getting Started](#getting-started)
7. [Deploying to a Custom Domain](#deploying-to-a-custom-domain)
8. [The Reviews Pipeline](#the-reviews-pipeline)
9. [The Practice Studio](#the-practice-studio)
10. [Progressive Web App](#progressive-web-app)
11. [Cost & Ownership](#cost--ownership)
12. [Browser Support](#browser-support)
13. [Roadmap Ideas](#roadmap-ideas)
14. [Credits](#credits)

---

## Overview

The Guitar Room's previous web presence was a static WordPress brochure site. This project replaces it with something genuinely useful: a fast, single-file website that doubles as an **installable practice app** for students, backed by a **live, self-refreshing Google Reviews pipeline** that costs nothing to run.

Every visitor — whether browsing on desktop, on a phone, or using the home-screen-installed app — gets the same red-and-graphite, logo-matched identity, the same course catalogue, and the same in-browser practice tools. There is no backend server to maintain, no database, and no recurring software bill.

**Design language:** warm light theme built from the school's own logo — brand red (`#c2182b`), graphite grey, and a stack of section tones (ivory, blush, sand, cool grey) so each part of the page reads as a distinct, intentional space rather than one long scroll. Typography pairs an editorial serif (Fraunces) with a clean grotesque (Hanken Grotesk) and a monospace accent (JetBrains Mono) for labels and data.

---

## Feature Tour

### For prospective students (the website)
- Full course catalogue across **Acoustic, Electric, and Ukulele**, each with a real technique curriculum
- Trinity College London certification pathway, explained
- Online lesson info (Zoom / Meet / Skype)
- Floating, always-visible **WhatsApp** and **Call** buttons that follow the page as you scroll
- A live **Google Reviews carousel** — real student reviews, rotating, with "Read more" on longer comments

### For enrolled students (the Practice Studio)
- **Reference Tuner** — two modes:
  - *Play tones*: tap a string to hear its exact pitch as a sustained reference tone
  - *Listen mode*: a real-time, **microphone-based pitch detector** that tells you sharp/flat with a needle gauge — built from scratch with no paid APIs
- **Metronome** — adjustable tempo, beats-per-bar (1–8), and rhythmic subdivisions (quarter/eighth/triplet/sixteenth)
- **Drum Machine** — a 16-step sequencer with six genres (Rock, Pop, Funk, Hip-Hop, 4-on-the-floor, Ballad), **3 playable variants per genre**, and a fully independent tempo and bar-length (3–8 bars) from the metronome — build a loop, then practice over it

### For the business
- A live reviews feed that **never needs manual updating** and is engineered to stay inside Google's free API tier indefinitely
- One shared codebase for the website *and* the installed app — no duplicate maintenance
- A custom-icon **installable app** (Android home screen / iOS "Add to Home Screen") that opens straight into the Practice Studio

---

## Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Markup/Styling | Hand-written HTML5 + CSS (no framework) | Single file, zero build step, loads instantly |
| Audio | Web Audio API (oscillators, noise buffers, `AnalyserNode`) | Every tone, drum hit, and pitch detection is **synthesized in-browser** — no licensed samples, no audio files, nothing to license |
| Fonts | Google Fonts (Fraunces, Hanken Grotesk, JetBrains Mono) | Free, open-license, web-served |
| Reviews backend | Cloudflare Worker + Workers KV | Free tier; serverless; no database to manage |
| Reviews source | Google Places API (Place Details) | Authoritative, real reviews — not invented content |
| Hosting | GitHub Pages | Free HTTPS hosting, custom domain support |
| App shell | Web App Manifest + Service Worker | Installable PWA — no app store, no app-store fees |

**Total recurring software cost: $0.** The only real-world costs anywhere near this project are *optional* and unrelated to running it: an Apple Developer account ($99/yr) or Google Play listing ($25 once) — and only if the project is ever repackaged for the app stores, which it currently is not.

---

## Project Structure

```
guitar-room-website/
├── index.html                  # The entire website + app (HTML, CSS, JS — one file)
├── manifest.webmanifest        # PWA install metadata (name, icons, colors, start URL)
├── sw.js                       # Service worker — caching strategy for offline/installed use
├── icon-192.png                # App icon, 192×192
├── icon-512.png                # App icon, 512×512
├── icon-maskable-512.png       # App icon with safe-zone padding for adaptive icon masks
├── apple-touch-icon.png        # iOS home-screen icon
└── tgr-reviews-worker.js       # Cloudflare Worker: live Google Reviews proxy + cache
```

The Cloudflare Worker is deployed separately (it runs on Cloudflare's infrastructure, not GitHub Pages) — see [The Reviews Pipeline](#the-reviews-pipeline).

---

## Architecture

```
┌──────────────────────────────┐
│         Visitor's Browser     │
│  (desktop, mobile, or the     │
│   installed home-screen app)  │
└───────────────┬───────────────┘
                │  HTTPS
                ▼
┌──────────────────────────────┐
│        GitHub Pages           │
│   theguitarroom.in            │
│   serves index.html + assets  │
└───────────────┬───────────────┘
                │  fetch() — reviews only
                ▼
┌──────────────────────────────┐
│      Cloudflare Worker        │
│   reads from KV cache first   │
└───────────────┬───────────────┘
                │  ~4 calls/day via Cron Trigger
                ▼
┌──────────────────────────────┐
│      Google Places API        │
│   Anna Nagar + Adambakkam     │
└──────────────────────────────┘
```

**Key design decision:** the website never calls Google directly, and a visitor never triggers a Google API call. A scheduled job (Cloudflare Cron Trigger, every 12 hours) is the *only* thing that ever talks to Google. Every page load reads from a pre-populated cache. This decouples cost from traffic entirely — the site could get 10 visitors or 10 million, and the Google API usage stays exactly the same.

---

## Getting Started

### Run it locally
This is a static, dependency-free site — there's no `npm install`, no build step.

```bash
git clone <this-repo>
cd guitar-room-website
# open index.html directly, or serve it locally:
python3 -m http.server 8000
```

> **Note:** PWA install prompts and the service worker only activate over HTTPS. A local `file://` or `localhost` preview will show all page content correctly, but won't show the install prompt — that requires a real deployed HTTPS origin (see below).

### Deploy (GitHub Pages)
1. Push all files in this repo to the `main` branch, with `index.html` at the **root**.
2. Repository → **Settings → Pages** → Source: **Deploy from a branch** → `main` / `(root)`.
3. Your site is live at `https://<username>.github.io/<repo>/` within a minute or two.

---

## The Reviews Pipeline

The reviews section shows **real Google reviews** from both studio locations (Anna Nagar and Adambakkam), refreshed automatically — without ever exposing an API key to the browser or risking a runaway bill.

### How it works
1. A **Cloudflare Worker** (`tgr-reviews-worker.js`) holds the Google API key server-side — it is never present in any client-facing code.
2. A **Cron Trigger** runs the worker every 12 hours. Each run fetches the latest reviews for both locations and merges them into a **rolling 6-month pool** stored in **Workers KV** — a small, durable key-value store.
3. The pool is deduplicated, capped (60 reviews), and filtered to drop anything over 6 sentences long, so the carousel stays readable.
4. Every page load reads directly from the KV cache — **zero Google API calls per visit.**
5. The website picks a **random subset** from the pool each refresh window (seeded by the cache's timestamp, so the same window always shows the same set, and it reshuffles on the next refresh) — prioritising reviewers who have a real profile photo.
6. If a scheduled refresh ever fails (e.g. a transient quota hiccup), the **last successfully cached set is served unchanged** — the site never falls back to placeholder content while a real cache exists.

### Cost safety, by design
- **Traffic-independent**: ~4 Google API calls per day, fixed, regardless of visitor count (12-hour cron × 2 locations).
- **Hard quota cap**: Google Cloud Console is configured with a low daily request quota (≈30/day) as a backstop — any runaway usage is rejected by Google before it can be billed.
- Realistic usage sits at roughly **120 calls/month**, comfortably inside Google's free monthly allowance.

### Setup
Full step-by-step deployment instructions (API key, KV namespace, binding, cron trigger) are documented in the header comment of `tgr-reviews-worker.js`.

---

## The Practice Studio

Every tool in the Practice Studio is built directly on the **Web Audio API** — no sample libraries, no third-party audio SDKs, nothing to license.

### Reference Tuner
- **Tone mode** plays a sustained, accurate reference pitch for each of the six standard strings (E A D G B E).
- **Listen mode** uses `getUserMedia()` and a custom **autocorrelation pitch detector** (the same family of algorithm used in commercial clip-on tuners) to listen to a live pluck and report sharp/flat in real time, with:
  - A low-pass filter stage to handle quieter or noisier microphones (notably phone mics) without misreading pitch
  - Median smoothing and a brief display "hold" so a decaying note reads as one steady result instead of flickering
  - A confidence check that withholds a reading rather than guessing when the signal is too ambiguous to trust

### Metronome
- 40–220 BPM, adjustable beats-per-bar (1–8), and four rhythmic subdivisions (quarter, eighth, triplet, sixteenth notes), with an accented downbeat.

### Drum Machine
- A 16-step sequencer per bar, **fully independent tempo and transport from the metronome** — they do not need to share a tempo or a click.
- Six genres, each with **3 distinct variants** (simple → busier), built from kick/snare/hi-hat conventions appropriate to the style.
- **3 to 8 bars** per loop, each bar individually editable, with a moving playhead that follows the groove across every bar as it loops.
- All drum sounds (kick, snare, closed/open hi-hat) are **synthesized live** from oscillators and filtered noise — there is no audio file or sample anywhere in the project.

---

## Progressive Web App

The site can be **installed** to a phone's home screen and runs full-screen, like a native app — without an app store.

- **Manifest** (`manifest.webmanifest`) defines the app name, icons, and theme color, and sets the install **start URL** to open straight into the Practice Studio.
- **App-mode detection**: when launched from the installed icon, the page detects standalone display mode and hides the marketing sections, presenting a focused practice tool instead — same codebase, different experience depending on how it was opened.
- **Service worker** (`sw.js`) caching strategy:
  - Page content: **network-first** — so edits to the live site reach the installed app on its next launch, with an offline fallback.
  - Icons/static assets: **cache-first** — fast, works offline, never re-downloaded unnecessarily.
  - The Reviews API: **never cached here** — always live, since it has its own purpose-built freshness system.

### Installing
- **Android (Chrome):** visit the site → an "Install app" prompt appears (or via the browser menu → *Install app*).
- **iPhone (Safari):** visit the site → **Share** → **Add to Home Screen**.

---

## Cost & Ownership

| Component | Cost |
|---|---|
| Website hosting (GitHub Pages) | Free |
| Fonts (Google Fonts) | Free |
| All audio (tuner, metronome, drum machine) | Free — synthesized in-browser, no licenses |
| PWA / installable app | Free — no app store fees |
| Cloudflare Worker + KV | Free tier |
| Google Places API (reviews) | ~$0 realistic usage, protected by quota cap |
| **Total** | **$0/month**, by design |

The only costs that could ever apply are entirely optional and outside this project's current scope: an Apple Developer Program membership ($99/year) or a Google Play one-time developer fee ($25), and only if this is ever repackaged for native app-store distribution.

---

## Browser Support

Built on standard, widely-supported web APIs (Web Audio, `getUserMedia`, Service Worker, Web App Manifest). Fully functional on current versions of Chrome, Safari, Firefox, and Edge, on both desktop and mobile. Microphone-based tuning requires a one-time permission grant in every browser, by design — this is a security requirement of the platform, not a project limitation.

---

## Roadmap Ideas

A few directions worth considering for a future iteration:

- **Swing/shuffle control** for the drum machine, for blues and shuffle feels
- **Tap-tempo** entry, so a tempo can be set by tapping along instead of dragging a slider
- A **calibration option** in the tuner for drop tunings or capo use
- A **standalone offline-only build** of the Practice Studio, for distribution independent of the marketing site, if ever needed
- A richer reviews widget (paid third-party service) if showing more than ~10 rotating reviews becomes a priority

---

## Credits

Designed and built for **The Guitar Room**, Chennai — Anna Nagar & Adambakkam studios.

Original brand identity (wordmark, color palette) preserved and extended throughout this redesign. Course curriculum, studio details, and Trinity College London certification information sourced from the school directly.

— *Built in collaboration with Claude (Anthropic).*
