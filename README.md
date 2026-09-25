# stalwart @ ynk.one

Self-hosted [Stalwart](https://stalw.art) mail server for `ynk.one`, deployed as a Temps
docker-compose project on `console.ynk.one` (Hetzner Cloud, 91.98.91.115).

- `docker-compose.yml` — the deployed stack (mail ports published on the host; admin UI on
  `127.0.0.1:18080` behind the Temps proxy at https://mail.ynk.one).
- `TUTO.md` — one-time setup steps that require the owner's accounts (Cloudflare, Namecheap,
  Hetzner) and secrets.

Config changes: edit `docker-compose.yml`, commit, push to `main` → Temps redeploys.
Mail data lives in the named volumes `stalwart-etc` / `stalwart-data` and survives redeploys.
