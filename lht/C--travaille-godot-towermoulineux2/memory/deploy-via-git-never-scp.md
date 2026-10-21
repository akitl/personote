---
name: deploy-via-git-never-scp
description: "The agent should deploy to the tipforgood test server via git push + ssh/docker on the server, never scp/rsync"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 7020066a-783a-415a-a50d-d8623080aee1
  modified: 2026-10-16T07:52:09.019Z
---

The agent should be able to update the tipforgood.com test server (not prod; its database backup does not matter). Code goes through git (`git push`), then the server is updated over `ssh` (git pull, docker / docker compose commands). Never scp (no direct file copy).

**Why:** stated by the user on 2026-10-16; CLAUDE.md updated accordingly (commits b7a217e, 1a48921 on v1).

**How to apply:** no scp/rsync, no docker push, no remote changes; prod only on explicit request. The auto-mode classifier refused editing `.claude/settings.json` (self-modification): the user has to remove the deny rules (`git push`, `ssh`, `*tipforgood.com*`, WebFetch tipforgood) themselves. Origin is github.com/SummoningSystems/towermoulineux2; origin/v1 was 95 commits behind local v1 on 2026-10-16; how the server is installed (host, path, compose) is not documented in the repo.
