# VodiWalker — Cloudflare Workers edition

This package adapts the supplied VodiWalker source to Cloudflare Python Workers
while keeping the existing FastAPI routes/UI/modules.

## 1) Cloudflare deployment

Use **Python Workers / pywrangler**, not the old Railway command.

Build/deploy command:
```bash
uv run pywrangler deploy
```

Local:
```bash
uv run pywrangler dev
```

If Cloudflare Git integration asks for a build command, use:
```text
uv run pywrangler deploy
```

Do **not** use:
```text
pip install -r requirements.txt
npx wrangler deploy
```

The old `requirements.txt`, `Dockerfile.txt`, and `railway.json` have been
removed from the deployment root so Cloudflare does not select the old Railway
Python build path.

## 2) Durable state — recommended

The Worker supports an optional KV binding named `VODI_KV`. When present, the
existing VodiWalker state is stored under:

```text
vodiwalker_state
```

Create a KV namespace:

```bash
npx wrangler kv namespace create VODI_KV
```

Take the returned namespace ID and add this to `wrangler.jsonc`:

```jsonc
"kv_namespaces": [
  {
    "binding": "VODI_KV",
    "id": "YOUR_NAMESPACE_ID"
  }
]
```

Then deploy again.

Without KV, the application can start and its API/UI work, but Worker
filesystem storage is ephemeral and must not be treated as durable storage.

## 3) Public URL

After deployment, open the panel and set VodiWalker's `public_base_url` to
the Worker URL, e.g.:

```text
https://your-worker.workers.dev
```

This keeps generated subscription/config URLs stable.

## 4) What was changed

- Added `src/main.py` as the Worker entrypoint.
- Added `Default = asgi.entrypoint(app)` for FastAPI.
- Added `wrangler.jsonc` with the `python_workers` compatibility flag.
- Added `pyproject.toml` for Python Worker dependencies.
- Removed the old Uvicorn launch block.
- Removed the old `requirements.txt` deployment path.
- Added Cloudflare request-environment handling.
- Added optional KV persistence without changing the existing state format.
- Added a safe fallback when `psutil` process telemetry is unavailable.
- Kept the existing UI, API routes, subscription/config generation, sales
  module, Telegram module, VLESS WebSocket relay and XHTTP module.

## 5) Important Cloudflare limitation

The original `tcp_relay.py` starts an **inbound raw TCP listener**. Cloudflare
Workers currently support inbound HTTP/HTTPS and WebSockets and outbound TCP,
but direct inbound TCP to a Worker is not currently available.

Therefore this build intentionally does not start the original raw TCP listener
when `CLOUDFLARE_WORKERS=1`. It does not pretend that `vless-tcp` has become a
native inbound Worker transport.

The existing HTTP/WebSocket routes remain available.

## 6) Verification performed on this package

- All Python modules compile successfully.
- The FastAPI application imports successfully.
- The application registered 83 routes in the supplied source.
- Non-parameterized GET routes were exercised locally; no HTTP 5xx responses
  were produced in that smoke test.
- Protected API routes correctly returned 401 when no authentication was
  supplied.
