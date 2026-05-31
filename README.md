# sevis.fi

[sevis.fi](https://sevis.fi) — personal website. Deployed automatically to Hetzner VPS via GitHub Actions on push to `main`.

## Structure

```
html/
├── index.html          # Main page (CV/portfolio)
├── terminal.css        # Shared stylesheet (nav, typography, mobile)
└── devlog/
    ├── index.html      # Devlog page (blog posts in <article> tags)
    └── healbot-screenshot.png
nginx/
└── sevis.fi.conf       # nginx server block (TLS via Certbot)
.github/workflows/
└── deploy.yml          # CI/CD pipeline
```

## How to Update

### Add a devlog post

1. Open `html/devlog/index.html`
2. Add a new `<article>` block before or after existing ones
3. Push to `main` — deploys automatically

### Add a new page/subpath

1. Create a directory under `html/` (e.g. `html/newpage/index.html`)
2. Push to `main` — nginx serves it at `sevis.fi/newpage/` via `try_files`

### Edit styles

All shared styles (nav, typography, responsive) live in `html/terminal.css`. Page-specific styles are in inline `<style>` blocks in each HTML file.

The nav is responsive:
- Desktop: fixed top-right
- Mobile (≤600px): relative, centered, wraps

### Add images

Drop them in the relevant directory (e.g. `html/devlog/`) and reference with an absolute path (`/devlog/image.png`).

## Pipeline

### What happens on push to `main`

1. **Clean** — SSH into VPS, `rm -rf /var/www/sevis.fi/html/*` (removes stale files)
2. **Copy** — SCP all files from `html/` to `/var/www/sevis.fi/html/`
3. **Reload** — `nginx -t && systemctl reload nginx`

### Secrets (GitHub repo settings)

| Secret | Purpose |
|--------|---------|
| `VPS_HOST` | Hetzner VPS IP |
| `VPS_USER` | SSH user |
| `VPS_SSH_KEY` | Private key for SSH |

### Important notes

- Deploys wipe the html directory first, so deleted files are actually removed from the server
- No build step — files are served as-is
- CSS may be cached by browsers after updates — hard refresh (Ctrl+Shift+R) to see changes
- TLS is managed by Certbot on the VPS (auto-renews)

## Server

- **Provider**: Hetzner VPS
- **Web server**: nginx
- **TLS**: Let's Encrypt via Certbot
- **OS**: (check VPS)
- **Site root**: `/var/www/sevis.fi/html/`
- **nginx config**: `/etc/nginx/sites-available/sevis.fi` (symlinked to sites-enabled)

## Adding a subdomain

1. Add DNS A record pointing to VPS IP
2. Add new nginx server block in `nginx/`
3. Run `certbot --nginx -d newdomain.sevis.fi` on the VPS
4. Update deploy workflow if needed
