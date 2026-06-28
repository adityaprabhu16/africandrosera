# africandrosera — gallery media proxy

Netlify site that proxies public gallery **image** URLs to the Pi Flask API.
Gives photo finders and scrapers an Africandrosera-branded hostname without
moving `africandrosera.org` DNS to Cloudflare.

```text
https://africandrosera.netlify.app/api/gallery/files/species/...
        ↓ (Netlify proxy)
https://canopyserver.alurellc.com/api/gallery/files/species/...
        ↓
Pi (Flask) → file on disk
```

Gallery UI stays on `droseragallery.netlify.app`. Uploads and admin API calls
still use `canopyserver.alurellc.com` directly.

Operator guide (monorepo): `canopysystemsui/gallery-app/READMES/GALLERY_MEDIA_HOST_README.md`

In the monorepo this repo is the **`africandrosera/`** git submodule (sibling of
`gallery-app`, `gallery-backend`).

---

## Deploy on Netlify

1. **GitHub:** push this repo (see below).
2. **Netlify:** Add new site → Import from Git → select **`africandrosera`** repo.
3. Build settings (auto-detected from `netlify.toml`):
   - Base directory: *(repo root)*
   - Build command: `echo '...'` (no-op)
   - Publish directory: `public`
4. **Site name:** `africandrosera` → live at `https://africandrosera.netlify.app`
5. Deploy.

No environment variables required on this site.

Push from monorepo root: `./scripts/push-africandrosera.sh -m "your message"`

---

## Enable branded image URLs

After Netlify deploy succeeds:

**Pi `.env`:**

```bash
GALLERY_PUBLIC_BASE="https://africandrosera.netlify.app"
```

Restart `gallery_app`:

```bash
sudo systemctl restart gallery_app
```

**Netlify `droseragallery` env:**

```bash
REACT_APP_GALLERY_IMAGE_BASE="https://africandrosera.netlify.app"
```

Redeploy gallery master.

Verify:

```bash
curl -I "https://africandrosera.netlify.app/api/gallery/files/species/<slug>/<file>.jpg"
```

---

## Latency and edge cache

- **First load** of each image URL: Netlify fetches from Pi via `canopyserver`
  (~100–400 ms extra vs direct tunnel).
- **Repeat loads:** cached at Netlify edge for **24 hours** when `gallery_app`
  returns:

  `Cache-Control: public, max-age=86400, stale-while-revalidate=604800`

  (set in `gallery-backend` `gallery_routes.py` — not from `netlify.toml` headers,
  which do not apply to proxied paths).

Verify after Pi deploy + second request:

```bash
curl -I "https://africandrosera.netlify.app/api/gallery/files/<path>.jpg"
# cache-status: "Netlify Edge"; hit  (on repeat)
```

---

## Later upgrade

Add `media.africandrosera.org` as a **custom domain** on this same Netlify site
(one CNAME from Alex) — no Pi tunnel DNS changes needed.
