# Deploy checklist — GitHub + Netlify

Local git repo is initialized and committed. `gh` CLI was not available in this
environment — create the GitHub repo manually (one time), then push.

---

## 1. Create GitHub repo

1. https://github.com/new
2. **Repository name:** `africandrosera`
3. **Public** (or private — Netlify can use either)
4. **Do not** add README, .gitignore, or license (already in local repo)
5. Create repository

---

## 2. Push from your Mac

```bash
cd /Users/adityaprabhu/Projects/canopysystemsui/africandrosera

git remote add origin https://github.com/adityaprabhu16/africandrosera.git
git branch -M main
git push -u origin main
```

(Use SSH remote if you prefer: `git@github.com:adityaprabhu16/africandrosera.git`)

---

## 3. Connect Netlify

1. https://app.netlify.com → **Add new site** → **Import an existing project**
2. **GitHub** → authorize → select **`africandrosera`**
3. Build settings (from `netlify.toml`):
   - Build command: (auto)
   - Publish directory: `public`
4. **Site name:** `africandrosera`
5. **Deploy site**

Live URL: `https://africandrosera.netlify.app`

---

## 4. Pi

```bash
# In production .env
GALLERY_PUBLIC_BASE="https://africandrosera.netlify.app"

sudo systemctl restart flask_app
```

---

## 5. Netlify `droseragallery`

Environment variables:

```bash
REACT_APP_GALLERY_IMAGE_BASE="https://africandrosera.netlify.app"
```

**Deploys** → **Trigger deploy** → **Deploy site**

---

## 6. Verify

```bash
curl -I "https://africandrosera.netlify.app/api/gallery/files/species/<slug>/<file>.jpg"
```

Open gallery → copy image address → should start with `https://africandrosera.netlify.app/`.

---

## Rollback

Clear `REACT_APP_GALLERY_IMAGE_BASE` on droseragallery and revert Pi
`GALLERY_PUBLIC_BASE` to `https://canopyserver.alurellc.com` if images break.
