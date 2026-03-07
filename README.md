# Rover Car Club of Australia — Members Portal & Website

A full-stack React web application providing a public website and private members portal for the RCCA.

---

## Quick Start (Local Development)

```bash
npm install
npm run dev
```

Then open http://localhost:5173 in your browser.

**Demo logins:**
- Admin: `admin@rcca.com.au` / `admin123`
- Member: `patricia@rcca.com.au` / `member123`

---

## Publishing to the Web (Netlify — Free)

This is the recommended way to share the site publicly. You can publish and unpublish at any time.

### Step 1 — Push to GitHub

1. Go to https://github.com and sign in (or create a free account)
2. Click **New repository** (the green button)
3. Name it `rcca-app`, set it to **Private** (only you can see the code)
4. Click **Create repository**
5. On your computer, open Terminal and run:

```bash
cd path/to/rcca-app        # navigate to this folder
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/rcca-app.git
git push -u origin main
```

### Step 2 — Deploy on Netlify

1. Go to https://netlify.com and sign in with your GitHub account
2. Click **Add new site → Import an existing project**
3. Choose **GitHub** and select your `rcca-app` repository
4. Netlify will auto-detect the settings from `netlify.toml`
5. Click **Deploy site**

Within 2 minutes your site will be live at a URL like:
`https://rcca-app-abc123.netlify.app`

### To make it public or private:

- **Public (default):** Anyone with the URL can view it
- **Password protect:** In Netlify → Site settings → Access control → Password protection
- **Take it offline:** In Netlify → Site settings → Danger zone → Disable site (instant)
- **Custom domain:** You can add `www.rovercarclubaust.com.au` or similar in Netlify settings

---

## Updating the Site

### If you're using Claude to make changes:

1. Download `src/App.jsx` from GitHub (or keep a local copy)
2. Upload it to a Claude conversation and ask for changes
3. Download the updated file
4. Replace `src/App.jsx` in your project
5. Run:

```bash
git add src/App.jsx
git commit -m "Update: describe what changed"
git push
```

Netlify will automatically redeploy within ~60 seconds.

---

## Project Structure

```
rcca-app/
├── src/
│   ├── App.jsx          ← The entire application (edit this file)
│   └── main.jsx         ← Entry point (don't need to touch this)
├── public/
│   └── favicon.svg      ← Browser tab icon
├── index.html           ← HTML shell (don't need to touch this)
├── package.json         ← Dependencies
├── vite.config.js       ← Build config
├── netlify.toml         ← Netlify deployment config
└── .gitignore
```

---

## Features

**Public Website**
- Home page with hero, stats bar, news, events preview
- About page with history, committee, membership info, contact
- News articles with full-text view
- Events listing

**Members Portal** (login required)
- Dashboard with activity summary
- Member management (admin)
- Vehicle register with transfer history
- Dues & renewals tracking (admin)
- Meetings with RSVP
- Events & shows with RSVP
- Kanban task board
- Marketplace classifieds
- Club shop with cart
- News admin (admin)
- **Site Content editor** — edit public page text without touching code (admin)
- My Profile

---

## Storage

Member data is stored in the browser's local storage (or Claude's artifact storage when running in Claude.ai). Each user's data is independent per browser/device. For a production deployment with shared data across members, a backend database would be needed.
