# zoom-seaweedfs

SeaweedFS (S3-compatible storage) for zoom-prokat, deployed via Coolify.

## Ports

| service              | port  | auth                                  |
|----------------------|-------|---------------------------------------|
| master + S3 API      | 8333  | S3 keys (identities in `base-config.json`) |
| master UI            | 9333  | none — do NOT expose publicly          |
| admin UI             | 23646 | login + password (`weed admin`)        |

## Environment variables

Set in Coolify (template: `.env.example`):

- `SERVICE_USER_S3` / `SERVICE_PASSWORD_S3` — S3 access key/secret (admin identity).
- `SERVICE_USER_ADMIN` / `SERVICE_PASSWORD_ADMIN` — admin UI login.
  Auth is enabled only when the password is non-empty (`-adminPassword` empty = no login form).

If a password contains `$`, `\` or `/`, the entrypoint `sed` may mangle it — prefer alphanumeric passwords.

## How interpolation works here (important)

`${...}` in docker-compose.yaml is resolved at **compose parse time** from Coolify's `.env`
(only `SERVICE_*` names exist there) — not from the container `environment:` block.
That is why `command:` must reference `${SERVICE_USER_ADMIN}` directly, and the master
entrypoint uses `$$AWS_ACCESS_KEY_ID` (escaped `$$` → `$` passed to the runtime shell,
which reads the var from the container env set from `${SERVICE_USER_S3}`).

`content:` under the bind mount is a Coolify extension (writes the file on the host);
plain `docker compose` rejects it — that validation error is expected locally.

## Local check

```bash
cp .env.example .env
docker compose --env-file .env config   # render & verify flags are non-empty
docker compose --env-file .env up       # run
```
