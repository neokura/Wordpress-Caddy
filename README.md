# Neokura — High-Traffic WordPress Stack (Mac mini M4)

Stack WordPress **optimisée pour un site média à très fort trafic**, hébergée sur **Mac mini M4**, avec **Cloudflare (plan gratuit)**, **Varnish**, **Caddy**, **Redis** et **Docker**.

Objectif : **tenir un trafic massif** sur une machine locale en déportant l’essentiel de la charge vers Cloudflare, sans fonctionnalités payantes ni mécanismes opaques.

---

## 🎯 Objectifs

- Supporter un **trafic très élevé**
- Performances maximales (PSI ≈ 100)
- Cache **prévisible et purgeable**
- Compatible **Cloudflare Free**
- Stack simple, lisible, maintenable

---

## 🧱 Architecture

Client → Cloudflare → Varnish → Caddy → WordPress → MariaDB / Redis

---

## 🐳 Services Docker

- wordpress — WordPress PHP-FPM  
- db — MariaDB  
- redis — Object Cache  
- varnish — Cache HTTP  
- caddy — Reverse proxy  
- wpcron — Cron externalisé  
- cloudflared — Tunnel Cloudflare  

WP-Cron est **désactivé** côté WordPress.

---

## ☁️ Cloudflare – Cache Rules (Free Plan)

### Rule 1 — Bypass WordPress sensible

(http.host eq "neokura.com" and (
  http.request.uri.path eq "/wp-login.php" or
  starts_with(http.request.uri.path, "/wp-admin") or
  starts_with(http.request.uri.path, "/wp-json") or
  http.request.uri.path eq "/wp-cron.php" or
  http.request.uri.path contains "/admin-ajax.php" or
  http.request.uri.path eq "/xmlrpc.php" or
  any(http.request.headers["cookie"][*] contains "wordpress_logged_in_") or
  any(http.request.headers["cookie"][*] contains "wordpress_sec_") or
  any(http.request.headers["cookie"][*] contains "wp-postpass_") or
  any(http.request.headers["cookie"][*] contains "comment_author_")
))

Action : **Bypass cache**

---

### Rule 2 — Assets statiques

(http.host eq "neokura.com" and (
  ends_with(http.request.uri.path, ".woff2") or
  ends_with(http.request.uri.path, ".woff") or
  ends_with(http.request.uri.path, ".css") or
  ends_with(http.request.uri.path, ".js") or
  ends_with(http.request.uri.path, ".png") or
  ends_with(http.request.uri.path, ".jpg") or
  ends_with(http.request.uri.path, ".webp") or
  ends_with(http.request.uri.path, ".svg")
))

- Cache Everything  
- Edge TTL : 1 an  
- Browser TTL : 1 mois  

---

### Rule 3 — HTML public

(http.host eq "neokura.com" and
 http.request.method in {"GET" "HEAD"} and
 http.request.uri.query eq "" and
 not starts_with(http.request.uri.path, "/wp-admin") and
 http.request.uri.path ne "/wp-login.php" and
 not starts_with(http.request.uri.path, "/wp-json")
)

- Cache Everything  
- Edge TTL : 120s  

---

## 🔁 Purge synchronisée

WordPress purge **Varnish + Cloudflare** automatiquement.

---

## 🔤 Fonts

Fonts auto-hébergées avec `font-display: swap` et cache long.

---

## 📈 Performances

- PSI Mobile : 100
- LCP ≈ 1.5s
- TBT : 0ms
- CLS : 0

---

## 🛠️ Commandes utiles

curl -sI https://neokura.com/ | egrep -i "cf-cache-status|age|via"

---

## 📄 Licence

Libre d’utilisation.
