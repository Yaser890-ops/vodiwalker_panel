# VodiWalker Cloudflare GitHub package

این نسخه برای Deploy از GitHub به Cloudflare Workers آماده شده است.

**Cloudflare Deploy Command:**

```bash
uv run pywrangler deploy
```

**Worker entry:** `src/main.py`

راهنمای کامل: `CLOUDFLARE-GITHUB-DEPLOY.md`


Cloudflare build note: the project version in pyproject.toml uses a PEP 440-compatible version (27.3.0). Workers Builds may run pip install . automatically before the deploy command.
