# VodiWalker — Cloudflare Python Worker

این نسخه برای Cloudflare Python Workers آماده شده است.

## Deploy

Cloudflare Build/Deploy command:

```bash
uv run pywrangler deploy
```

Local:

```bash
uv run pywrangler dev
```

`npx wrangler deploy` را برای این نسخه استفاده نکنید؛ Python Worker باید با `pywrangler` بسته‌بندی و deploy شود.

## Required secrets

در Cloudflare Secret/Variableهای زیر را تنظیم کنید:

- `ADMIN_USERNAME` — پیش‌فرض `admin`
- `ADMIN_PASSWORD` — رمز ورود را حتماً تغییر دهید
- `SECRET_KEY` — یک مقدار تصادفی بلند

## Optional KV

برای ماندگاری state یک KV binding با نام `VODI_KV` بسازید.

## Cloudflare limitation

Worker ورودی TCP خام ندارد. بنابراین VLESS TCP relay خام این نسخه روی Worker اجرا نمی‌شود. HTTP/WebSocket APIها و پنل از مسیر Worker اجرا می‌شوند. Cloudflare در حال حاضر TCP خروجی را با `connect()` ارائه می‌کند، اما inbound TCP هنوز پشتیبانی نمی‌شود.

## Cloudflare-native deployment (updated)

برای GitHub/Workers Builds از دستور زیر استفاده کنید:

`uv run pywrangler deploy`

حتماً `SECRET_KEY`، `ADMIN_USERNAME` و `ADMIN_PASSWORD` را در Cloudflare تنظیم کنید. برای ماندگاری state، KV binding با نام `VODI_KV` اضافه کنید.

APIهای `/api/network/cloudflare` و `/api/network/http-check` برای تشخیص و تست قابلیت‌های Cloudflare اضافه شده‌اند.

## Gemini / Clean IP Mode

The panel now includes `Gemini / Clean IP Mode`.

- The country selector is restricted to countries currently listed by Google for Gemini availability.
- `GET /api/gemini/countries` exposes the list.
- `GET /api/gemini/clean-ips?country=US&limit=20` retrieves country-tagged Cloudflare/CF-proxy candidates from the configured source, with a GitHub country-list fallback.
- `POST /api/gemini/check` performs a best-effort HTTPS reachability test against a candidate using the selected Host/path and returns latency/status.
- Successful candidates can be copied into the panel's Clean IP list and used to generate multiple configs.

Important: Cloudflare's normal proxied IP ranges are shared Anycast addresses, so a Cloudflare IP is not inherently a permanent country-specific endpoint. The country label is therefore treated as a candidate filter, not a guarantee of Gemini availability. Google availability is also separate from whether a particular IP/route is accepted at a given moment.

Default candidate source:
`https://ipdb.api.030101.xyz/?type=bestcf&country=true`

Fallback country files:
`https://raw.githubusercontent.com/cmliu/cloudflare-better-ip/main/{COUNTRY}-443.txt`

Override the primary source with the `GEMINI_IP_SOURCE` environment variable if desired.
