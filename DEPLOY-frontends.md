# KidoMatic — Frontend Deployment

Static builds (already built with `VITE_API_URL=https://api.kidomatic.app/api`).
Nothing to build on the server — just copy files + nginx + certbot.

Subdomains:
- `kidomatic.app` + `www` → landing site
- `school.kidomatic.app` → school ERP, lands on staff login (`/app/login`)
- `parent.kidomatic.app` → same ERP build, lands on parent login (`/portal/login`)
- `app.kidomatic.app` → SaaS admin

All five A-records already point to the instance. API is already live.

## 1. Upload + unzip
From your laptop:
```bash
scp kidomatic-frontends.zip ubuntu@16.178.49.100:/tmp/
```
On the server:
```bash
cd /tmp && unzip -o kidomatic-frontends.zip
```

## 2. Put the builds in place
```bash
sudo mkdir -p /var/www/kidomatic-landing /var/www/kidomatic-erp /var/www/kidomatic-saas
sudo cp -r /tmp/kidomatic-frontends/dist/landing/* /var/www/kidomatic-landing/
sudo cp -r /tmp/kidomatic-frontends/dist/erp/*     /var/www/kidomatic-erp/
sudo cp -r /tmp/kidomatic-frontends/dist/saas/*    /var/www/kidomatic-saas/
```

## 3. Install the nginx sites
```bash
sudo cp /tmp/kidomatic-frontends/nginx/*.conf /etc/nginx/sites-available/
sudo ln -sf /etc/nginx/sites-available/kidomatic.app.conf        /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/school.kidomatic.app.conf /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/parent.kidomatic.app.conf /etc/nginx/sites-enabled/
sudo ln -sf /etc/nginx/sites-available/app.kidomatic.app.conf    /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```
(The existing `api.kidomatic.app` site stays untouched.)

## 4. SSL for all four subdomains (one command)
```bash
sudo certbot --nginx \
  -d kidomatic.app -d www.kidomatic.app \
  -d school.kidomatic.app -d parent.kidomatic.app -d app.kidomatic.app \
  --non-interactive --agree-tos -m ravi@hackgenix.com --redirect
```

## 5. Verify (from your laptop)
- https://kidomatic.app        → landing
- https://school.kidomatic.app → redirects to /app/login (staff)
- https://parent.kidomatic.app → redirects to /portal/login (parent)
- https://app.kidomatic.app    → SaaS admin login

Logins (seeded, password `demo1234`): principal@bluelagoon.edu,
parent@bluelagoon.edu, superadmin@juniorocto.com. **Rotate before public launch.**

## Updating a frontend later
Rebuild locally (`npm run build`), copy the new `dist/*` over the matching
`/var/www/...` folder, done — no nginx/cert changes:
```bash
sudo cp -r dist/* /var/www/kidomatic-erp/     # example
```

## Notes
- `school.` and `parent.` serve the SAME files (`/var/www/kidomatic-erp`); only
  the `/` redirect differs. That's intentional — one ERP build, two entry points.
- If you later want the reseller console, say so — it's the same pattern on
  `reseller.kidomatic.app`.
- CORS is already set on the API for all these origins.
