---
name: tipforgood-lht2v1-deploy
description: "How the LHT2 v1 test server on tipforgood.com is deployed (ssh port, path, compose service, nginx route)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7020066a-783a-415a-a50d-d8623080aee1
  modified: 2026-10-16T08:03:47.734Z
---

tipforgood.com = 77.37.120.101, ssh `benoit@77.37.120.101 -p 2222` (port 22 times out). Shared host: other apps (comptaos, carton2000, psy2000 prod, tipforgood front, lht2test) must not be touched.

- `https://tipforgood.com/lht2v1/` → nginx → 172.17.0.1:8083 → container `lht-2-server-1`, compose dir `~/apps/lht2` (checkout of branch `v1`, tracks origin/v1 over git@github; untracked `docker-compose.override.yml` maps the 8083 port).
- `/lht2/api/` → 8082 → `lht2test-server-1` in `~/apps/lht2-test` (older stack, not the v1 one).
- Deploy recipe (done 2026-10-16 for 1a48921, then 79c5832 after wave 6): `git push origin v1` locally; then ssh: `cd ~/apps/lht2 && git pull --ff-only && docker compose build server && docker compose up -d --no-deps server`; check `docker logs lht-2-server-1` for `migration applied` / `server started`, and `curl https://tipforgood.com/lht2v1/health/ready` (database + engine ok).
- The auto-mode classifier refused reading `.env` on the server; don't read it.

Related: [[deploy-via-git-never-scp]]
