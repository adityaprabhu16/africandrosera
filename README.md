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

---

## Enable branded image URLs

After Netlify deploy succeeds:

**Pi `.env`:**

```bash
GALLERY_PUBLIC_BASE="https://africandrosera.netlify.app"
```

Restart Flask/gunicorn.

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

## Latency

- **First load** of each image: ~100–400 ms extra (Netlify fetches from Pi).
- **Repeat loads:** cached at Netlify edge (24h `Cache-Control`).

---

## Later upgrade

Add `media.africandrosera.org` as a **custom domain** on this same Netlify site
(one CNAME from Alex) — no Pi tunnel DNS changes needed.
