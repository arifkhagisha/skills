# NoDrop Mail

Temp mail pribadi, receive-only. Kode OTP tampil langsung di daftar inbox,
tanpa membuka pesan.

## Isi repo

- `cloudflare-worker/src/index.js` — Worker Cloudflare: Email Routing
  catch-all → simpan ke KV, API inbox terkunci token, frontend terintegrasi,
  ekstraksi OTP (angka, huruf, campuran), baca inbox tanpa KV list.
- `cloudflare-worker/wrangler.toml` — template konfigurasi Worker
  (isi DOMAINS dan KV ID sendiri, lihat placeholder di file).
- `vercel-frontend/index.html` — frontend yang di-deploy ke Vercel,
  API-nya ke Worker Cloudflare.

## Konfigurasi (diatur di dashboard, TIDAK ada di repo ini)

- Cloudflare: Worker, KV `TEMPMAIL_KV`, `DOMAINS`, `MESSAGE_TTL_SECONDS=3600`,
  secret `ACCESS_TOKEN`, Email Routing catch-all → Worker.
- Vercel: deploy `vercel-frontend/index.html` sebagai production.
- DNS: MX domain → Email Routing Cloudflare, subdomain API → Worker route,
  subdomain web → domain Vercel.

## Catatan keamanan

Jangan pernah commit nilai `ACCESS_TOKEN`, API Token Cloudflare, atau KV ID
asli ke repo ini. Token hanya diinput lewat UI dan disimpan di browser
pengguna, atau sebagai secret di dashboard Cloudflare.
