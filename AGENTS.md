# AGENTS.md

Single-node MinIO (community image) local deployment via `docker-compose.yml`. No source code, tests, build, or CI — the repo is a container-run shell around one compose file.

## Setup before any `docker compose up`

1. `cp .env.example .env` then set `MINIO_ROOT_USER` (≥3 chars) and `MINIO_ROOT_PASSWORD` (strong). Without `.env`, compose will pass empty credentials and the server will reject startup with `MINIO_ROOT_USER length is 0`.
2. `.env` is gitignored — never commit. Real credentials live only in `.env`.

The repo root may still contain a leftover `minio.license` file (gitignored). It is **not** referenced by the current compose file and has no effect. Safe to leave or `rm` it.

## Commands

```bash
docker compose up -d       # start
docker compose logs -f minio
docker compose stop
docker compose down        # keep data/
docker compose down -v     # also drops named volumes (./data on host is NOT removed; rm -rf data/ manually)
```

## Port & endpoint facts (non-obvious)

- Host `9100` → container `9000` = S3 API. Use `--endpoint-url http://localhost:9100` with `aws`/`mc`/SDKs.
- Host `9101` → container `9101` = web console. Same port number inside and out by coincidence.
- Ports bind `0.0.0.0`; reachable from the LAN. For localhost-only, edit `docker-compose.yml` to `127.0.0.1:9100:9000` / `127.0.0.1:9101:9101`.
- Data volume is a host bind `./data:/mnt/data`, not a Docker named volume. Backups and cleanup operate on `./data` directly; `docker compose down -v` does not touch it.

## Image

`minio/minio:latest` — community MinIO build from Docker Hub. `latest` is floating; pin a digest if reproducibility matters.

> Note: this was migrated off `quay.io/minio/aistor/minio:latest` (AIStor build) because Quay's blob CDN was unusable from CN — small layers downloaded but the two big blobs (~83MB + ~31MB) endlessly retried with no error. The AIStor `--license` flag and `minio.license` bind-mount were removed at the same time. If you switch back to AIStor, you must re-add both.

## Conventions

- README is the source of truth for setup prose; verify any guidance against `docker-compose.yml` before acting (compose is the executable truth).
- `data/`, `.omo/`, `.DS_Store` are gitignored runtime/local state — do not commit.
- All human-facing docs are in Simplified Chinese; match that language when editing README or writing user-visible messages.
