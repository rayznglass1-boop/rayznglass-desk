# Nickalas Backend Security Policy

## Key Rotation (do this now if any secret was ever pasted in chat)

1. **Fernet key** — generate a fresh one:
   ```bash
   python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
   ```
   Update `docker.env` on every machine that runs the stack, restart containers.

2. **Supabase service role key** — rotate in the Supabase dashboard under Project Settings → API. The service role bypasses RLS; treat it as root.

3. **Supabase DB password** — rotate in Project Settings → Database. Update `docker.env`.

4. **GitHub PAT** — if a token was exposed, revoke it at github.com/settings/tokens and issue a new fine-scoped one. Store via `gh auth login` or a secret manager, never in files.

## Rules

- `.env` and `docker.env` are gitignored. Only `.env.example` (placeholders) is committed.
- RLS on `kv_store` is deny-by-default; only `service_role` may write.
- Health endpoints (`/health`, `/core/status`, `/supabase/health`) expose no secrets — safe to probe externally.
- Prefer Docker secrets or runtime injection over env files on any host that isn't air-gapped.

## Verify after rotation

```bash
docker compose up --build -d
curl -s localhost:8080/health
curl -s localhost:8080/core/status
curl -s localhost:8080/supabase/health
```

All three should return 200.