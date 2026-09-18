# VodiWalker — Cloudflare Worker deployment

این نسخه برای **Cloudflare Python Workers** آماده شده است.

## 1) Build/Deploy command

در Cloudflare Workers Builds اگر پروژه از GitHub ساخته می‌شود، دستور Deploy را این بگذارید:

```bash
uv run pywrangler deploy
```

از `npx wrangler deploy` برای این نسخه استفاده نکنید؛ `pywrangler` وابستگی‌های Python Worker را bundle می‌کند.

## 2) Variables / Secrets

این مقادیر را در Cloudflare به صورت Secret/Variable تنظیم کنید:

- `ADMIN_USERNAME` = نام کاربری پنل (پیشنهاد: admin)
- `ADMIN_PASSWORD` = رمز پنل
- `SECRET_KEY` = یک مقدار تصادفی طولانی و ثابت

برای `SECRET_KEY` حتماً مقدار ثابت قرار دهید تا session و hashها بعد از restart تغییر نکنند.

## 3) KV برای ذخیره دائمی

یک Workers KV Namespace بسازید و با نام binding زیر به Worker وصل کنید:

```text
VODI_KV
```

پنل به صورت خودکار state را در کلید `vodiwalker_state` ذخیره/بازیابی می‌کند.

بدون KV، نسخه Cloudflare ممکن است در restart/deployment state دائمی نداشته باشد.

## 4) قابلیت‌های شبکه Cloudflare

- HTTP/HTTPS check: فعال
- WebSocket endpoint: فعال
- outbound TCP برای relay: از socketهای Worker/Python Worker استفاده می‌شود
- inbound TCP مستقیم به Worker: پشتیبانی نمی‌شود و Cloudflare هم فعلاً آن را ارائه نمی‌کند
- Railway TCP Proxy: فقط روی Railway معنی دارد و در Cloudflare فعال نمی‌شود
- telemetry وابسته به psutil/filesystem: روی Cloudflare با metrics سطح برنامه جایگزین شده است

## 5) APIهای تشخیص Cloudflare

پس از ورود:

```text
GET /api/network/cloudflare
POST /api/network/http-check
GET /api/system/diagnostics
```

`/api/network/cloudflare` نشان می‌دهد KV و Secret و قابلیت‌های شبکه Worker در دسترس هستند.

## 6) نکته VLESS/XHTTP

VLESS WebSocket و XHTTP در این نسخه از socketهای خروجی Python Worker استفاده می‌کنند و تنظیمات سطح پایین socket که مخصوص Linux/Railway بودند در Cloudflare اجرا نمی‌شوند.

Cloudflare **TCP ورودی مستقیم** به Worker را پشتیبانی نمی‌کند؛ بنابراین پروتکل‌های خامی که انتظار `listen()` روی یک TCP port دارند نباید به عنوان TCP inbound واقعی Cloudflare معرفی شوند.
