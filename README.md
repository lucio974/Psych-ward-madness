# Psych Ward Madness — PWA

Low-poly 3D mobile brawler. Installable, fullscreen, works offline.

## Files

| File | Purpose |
|---|---|
| `index.html` | The game |
| `three.min.js` | three.js r128, bundled for offline play |
| `manifest.webmanifest` | App name, icons, fullscreen display |
| `sw.js` | Service worker (offline cache) |
| `icons/` | App icons (192, 512, maskable 512, Apple touch 180) |

## Requirements

- Served over **HTTPS** (or `localhost`) — service workers and install don't work on plain HTTP.
- Serve the folder as static files. No backend.

## Test locally

```
cd psych-ward-madness-pwa
python3 -m http.server 8080
```

Open `http://localhost:8080`.

## Deploy — Caddy (auto HTTPS)

Point a DNS A record (e.g. `pwm.yourdomain.com`) at the server first.

```
scp psych-ward-madness-pwa.zip root@YOUR_VPS_IP:/tmp/
ssh root@YOUR_VPS_IP
apt update && apt install -y unzip caddy
mkdir -p /var/www/pwm && unzip -o /tmp/psych-ward-madness-pwa.zip -d /var/www/pwm
cat > /etc/caddy/Caddyfile <<'CADDY'
pwm.yourdomain.com {
    root * /var/www/pwm
    file_server
    header /sw.js Cache-Control "no-cache"
}
CADDY
systemctl reload caddy
```

## Deploy — nginx + certbot

```
mkdir -p /var/www/pwm && unzip -o /tmp/psych-ward-madness-pwa.zip -d /var/www/pwm
cat > /etc/nginx/sites-available/pwm <<'NGINX'
server {
    server_name pwm.yourdomain.com;
    root /var/www/pwm;
    types { application/manifest+json webmanifest; }
    location = /sw.js { add_header Cache-Control "no-cache"; }
}
NGINX
ln -s /etc/nginx/sites-available/pwm /etc/nginx/sites-enabled/pwm
nginx -t && systemctl reload nginx
certbot --nginx -d pwm.yourdomain.com
```

## Install on a phone

- **Android / Chrome:** open the site, tap **INSTALL** on the title screen (or menu → Install app).
- **iPhone / Safari:** Share → **Add to Home Screen**.

## Updating

1. Replace the files in the web root.
2. Bump the cache name in `sw.js` (`const V='pwm-v23'` → `pwm-v24`).
3. Players get the new version on their next launch (close and reopen the app once).

## Saved data

Progress (best score, awards, wardrobe, name, room, camera choice) is stored in the browser's `localStorage` on each device. Clearing site data resets it.

## Controls

- **Touch:** left thumb = move, SWING (hold to charge), DASH, GRAB/THROW.
- **Keyboard:** WASD move · Space swing (hold to charge) · Shift dash · E grab/throw · C camera.
