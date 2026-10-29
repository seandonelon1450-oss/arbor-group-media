# AGENTS.md — arbor-group-media

## What this repository is
A media asset and legal-record library. There is **no application code**: no package.json, no
server, no build step, no database, no tests. Contents:

- `videos/` — 11 mp4 shorts (~55 MB total)
- `logos/` — 4 webp brand marks
- `bots/` — 16 agent portrait stills (jpg/webp)
- `legal/` — markdown/txt legal records
- `TIMESTAMP-MANIFEST-2026-10-19.md` — code-authorship manifest

Because there is nothing to compile, the Base44 environment serves the checked-out files with
nginx. `index.html` is the gallery UI (one file, hand-written, no dependencies).

## Running it
```
docker compose -f docker-compose.base44.yml up -d
```
- Serves on host port **3000**; the container is `nginx:alpine` with every asset folder
  bind-mounted read-only — edit `index.html` or add a file and reload, no rebuild needed.
- `.git` is intentionally **not** mounted, so repository history is never served. Keep it that way.
- `deploy/nginx.conf` renders `.md`/`.txt` inline as `text/plain` and lets nginx handle byte
  ranges on `/videos/` so video seeking works.

## Gotchas
- Adding a new asset folder means adding a matching bind mount in `docker-compose.base44.yml`
  (mounts are per-directory so `.git` stays hidden), then `up -d` again.
- `index.html` lists assets explicitly; a newly added file won't appear in the gallery until its
  card is added.
- Videos are large; the healthcheck probes `/index.html` only, so it stays fast.

## How to verify it works
- `curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/` → `200`
- `curl -sI http://localhost:3000/videos/<file>.mp4 | grep -i content-type` → `video/mp4`
- `curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/.git/config` → `404` (must stay blocked)
- `docker compose -f docker-compose.base44.yml ps` → `site` is `healthy`
