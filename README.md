# 📚 UniNotes — Amity Notes Portal

UniNotes is an academic resource portal built for Amity University students and teachers. Teachers upload course notes, students browse and download them by course and semester, and admins manage users, materials, and site-wide announcements — all from a single-page app with offline support.

**Status:** v5.2 — Production Ready

---

## ✨ Features

UniNotes supports four roles — **Guest**, **Student**, **Teacher**, and **Admin** — each enforced both client-side (`requireRole()` guard) and server-side (Supabase Row Level Security).

**Authentication**
- Email/password sign-in and sign-up with client-side validation
- Three-tier role resolution (Supabase SDK → REST fallback → RPC fallback)
- Rate-limited sign-in (5 requests / 5 minutes), password recovery via email
- Teacher accounts require admin approval before access is granted

**Student**
- Browse and search notes by course and semester, table or card view, pagination
- Download notes via signed URLs (60-minute expiry) with a per-file download cooldown
- Manage personal profile, view announcements
- Offline mode with a cached notes list

**Teacher**
- Upload PDF notes with strict validation (extension, MIME type, magic bytes, 50 MB max)
- Upload progress bar, duplicate-submit guard, automatic orphan storage cleanup on failed inserts
- Edit/delete own materials, create and manage announcements

**Admin**
- Full user management (approve/suspend teachers), searchable user list
- Manage all uploaded materials across every course
- Site-wide announcements
- Analytics dashboard (subject-wise uploads, download activity) via Chart.js, with proper chart instance cleanup

**Progressive Web App**
- Installable with a web manifest and shortcuts (My Notes, Assignments)
- Service worker with app-shell precaching, network-first/cache-first strategies, and an offline fallback page

---

## 🛠 Tech Stack

UniNotes is a vanilla single-page application — no frontend framework. The only npm usage is in the build pipeline.

| Layer | Technology |
|---|---|
| Structure | HTML5 (single `index.html`, CSS-toggled role views, semantic + ARIA, CSP-safe with no inline handlers) |
| Styling | CSS3 — hand-authored, custom properties for dark/light theming |
| Logic | JavaScript (ES2020), vanilla — `script_improved.js` |
| Backend | [Supabase](https://supabase.com) — Auth, PostgreSQL (via REST), Storage, RPC, Row Level Security |
| Charts | [Chart.js](https://www.chartjs.org/) 4.4.0 (CDN) |
| Auth/DB client | [@supabase/supabase-js](https://github.com/supabase/supabase-js) 2.50.0 (CDN) |
| Fonts | Syne (headings), DM Sans (body) — Google Fonts |
| Hosting | Netlify |
| Build | Node.js (built-in `fs`/`crypto` only — no npm packages required) |

---

## 📁 Project Structure

```
.
├── index.html          # App shell — all four role views in one file
├── script_improved.js  # Application logic (auth, routing, data, PWA, security)
├── style.css            # Full stylesheet, theming, layout, animations
├── sw.js                # Service worker (offline caching)
├── sw-register.js       # Service worker registration (no-cache, separate from FOUC script)
├── manifest.json        # PWA manifest
├── build.js             # Production build script — injects env vars + CSP hash
├── netlify.toml         # Netlify build config, redirects, and security headers
├── icon-192.png / icon-512.png
└── SETUP_CHECKLIST.md   # Pre-launch checklist (Supabase + Netlify setup)
```

---

## 🔒 Security

- **Content Security Policy** — script-src is hash-only (no `unsafe-inline`); style-src allows inline out of necessity for the dynamic stylesheet
- `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, HSTS (2-year, preload), strict referrer policy, and a locked-down Permissions-Policy
- Supabase credentials are **never** committed — they're injected at build time from environment variables into placeholder tokens (`%%SUPABASE_URL%%`, `%%SUPABASE_ANON_KEY%%`)
- Row Level Security enabled on all core tables (`profiles`, `materials`, `announcements`, `downloads`)
- Private storage bucket — files are only reachable via signed URLs
- Upload validation on file extension, MIME type, and PDF magic bytes

⚠️ **Never deploy the source folder directly.** It contains unresolved `%%...%%` placeholders — always run the build step first.

---

## 🚀 Getting Started

### Prerequisites
- Node.js (for the build script only — no packages to install)
- A [Supabase](https://supabase.com) project with the `profiles`, `materials`, `announcements`, and `downloads` tables set up, RLS policies applied, and a private `notes` storage bucket

### Local build

1. Create a `.env` file in the project root:
   ```
   SUPABASE_URL=https://your-project.supabase.co
   SUPABASE_ANON_KEY=your-anon-key
   ```
2. Run the build:
   ```bash
   node build.js --local
   ```
3. Serve the generated `dist/` folder with any static file server.

### Deploying to Netlify

1. Set `SUPABASE_URL` and `SUPABASE_ANON_KEY` in **Netlify → Site Settings → Environment Variables**.
2. Build command: `node build.js` · Publish directory: `dist`
3. Push to your connected repo, or drag-and-drop the locally built `dist/` folder.

See [`SETUP_CHECKLIST.md`](./SETUP_CHECKLIST.md) for the full pre-launch checklist (Supabase RLS, auth rate limits, storage bucket privacy, and Netlify config).

---

## 🗺 Roadmap

Planned improvements beyond v5.2 are tracked internally and are not required for the current version to function — see the project's technical reference document for the full list.

---

## 📄 License

Add a license of your choice (e.g. MIT) if you intend to open-source this project.
