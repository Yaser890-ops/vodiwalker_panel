# VodiWalker — Cloudflare GitHub Deploy

این بسته برای **Cloudflare Workers + Python Workers + GitHub Integration** آماده شده است.

## ساختار

- Entry point: `src/main.py`
- Wrangler config: `wrangler.jsonc`
- Python dependencies: `pyproject.toml`
- Deploy command: `uv run pywrangler deploy`
- Worker mode: `python_workers`

## اتصال GitHub به Cloudflare

1. این پروژه را در یک GitHub repository قرار دهید.
2. در Cloudflare بروید به:
   **Workers & Pages → Create application → Import a repository**
3. GitHub repository را انتخاب کنید.
4. Root directory را ریشه همین repository بگذارید.
5. اگر Cloudflare از شما Build command و Deploy command خواست:

### Build command

خالی بگذارید.

### Deploy command

```bash
uv run pywrangler deploy
```

6. Save and Deploy را بزنید.

Cloudflare با GitHub Integration بعد از pushهای بعدی می‌تواند Worker را دوباره build/deploy کند.

## KV

برای ذخیره دائمی state:

1. Workers & Pages → KV → Create namespace
2. سپس Worker → Settings → Bindings
3. KV namespace را با Binding Name زیر متصل کنید:

```text
VODI_KV
```

بعد از ساخت KV، در `wrangler.jsonc` مقدار ID واقعی namespace را اضافه کنید:

```json
"kv_namespaces": [
  { "binding": "VODI_KV", "id": "YOUR_REAL_KV_NAMESPACE_ID" }
]
```

اگر هنوز KV ندارید، Worker بدون این binding هم deploy می‌شود، ولی state دائمی نخواهد بود.

## Secret

در Worker → Settings → Variables and Secrets یک Secret بسازید:

```text
SECRET_KEY
```

برای آن یک مقدار تصادفی طولانی قرار دهید.

در این بسته هیچ secret واقعی داخل GitHub قرار داده نشده است.

## Admin

اگر بخش احراز هویت فعال پروژه به این متغیرها نیاز داشته باشد، آنها را به صورت Secret/Variable تنظیم کنید:

```text
ADMIN_USERNAME
ADMIN_PASSWORD
```

## مهم

این پروژه Python Worker است؛ آن را با `uvicorn`، `python src/main.py` یا یک TCP server معمولی اجرا نکنید.

برای Python Workers، Cloudflare از `pywrangler` برای bundle کردن Python dependencies استفاده می‌کند.

## بعد از Deploy

آدرس Worker معمولاً به شکل زیر خواهد بود:

```text
https://YOUR-WORKER.YOUR-SUBDOMAIN.workers.dev
```

برای تست APIها:

```text
GET /api/system/diagnostics
GET /api/network/cloudflare
GET /api/resources/sources
GET /api/gemini/countries
```

## محدودیت شبکه Cloudflare

Worker می‌تواند HTTP/HTTPS و WebSocket را ارائه کند و برای برخی ارتباطات outbound از قابلیت‌های Worker استفاده کند؛ اما Worker یک TCP listener عمومی معمولی مثل VPS/Railway نیست. بنابراین این پروژه ادعا نمی‌کند که روی Cloudflare یک TCP inbound خام روی پورت دلخواه ایجاد می‌کند.

اسکن واقعی IP:Port در این نسخه از Check-Host distributed TCP probe استفاده می‌کند و latency نمایش‌داده‌شده latency نودهای تست اینترنتی است، نه ping مستقیم موبایل کاربر.


Cloudflare build note: the project version in pyproject.toml uses a PEP 440-compatible version (27.3.0). Workers Builds may run pip install . automatically before the deploy command.


## Cloudflare startup-randomness fix
The Worker no longer generates SECRET_KEY during module import. Cloudflare Python Workers execute and snapshot top-level code during deployment, where OS randomness is intentionally unavailable. Set `SECRET_KEY` as a Worker secret; the application loads it lazily after the Worker request lifecycle starts.
