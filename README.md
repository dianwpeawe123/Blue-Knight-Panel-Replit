# Blue Knight Gate — Replit Edition

پنل VLESS / VMess / Trojan / XHTTP روی Replit — **ایمپورت کن و Publish کن. هیچ متغیری لازم نیست.**

A VLESS / VMess / Trojan / XHTTP panel for Replit — **import and publish. No variables required.**

## فارسی

### راه‌اندازی
1. توی Replit: **Create App → Import from GitHub** → این ریپو.
2. **Publish** کن (Reserved VM یا Autoscale). آدرس `https://<name>.replit.app` ساخته می‌شود.
3. برو به `https://<name>.replit.app/knight` و رمز پنل را بساز.
4. کانفیگ‌ها یا لینک اشتراک را کپی کن. همه روی پورت **443** و با **TLS** هستند.

پورت، دامنه و محل ذخیره خودکار تشخیص داده می‌شوند.

### اگر `replit.app` فیلتر است
یک **Cloudflare Worker** بساز که همهٔ درخواست‌ها را به `<name>.replit.app` بفرستد و این هدر را بگذارد:

`headers.set("X-Forwarded-Host", new URL(request.url).hostname)`

بعد پنل را از آدرس Worker باز کن (`https://<worker>.workers.dev/knight`). پنل همان دامنه را برای آدرس، SNI و Host همهٔ کانفیگ‌ها ذخیره می‌کند و دیگر لازم نیست لینک را دستی عوض کنی. بازدید مستقیم از `replit.app` این دامنه را عوض نمی‌کند. اگر Secret به نام `DOMAIN` بگذاری، همان اولویت دارد.

نمونهٔ کامل Worker (مسیرها را عوض نکن؛ پنل، `app.css`، `app.js`، `/health` و API مصرف همه باید رد شوند):

```js
export default {
  async fetch(request) {
    const url = new URL(request.url);
    const ORIGIN = 'https://<name>.replit.app'; // آدرس منتشرشدهٔ Replit

    const target = new URL(url.pathname + url.search, ORIGIN);
    const headers = new Headers(request.headers);
    headers.delete('host');
    headers.set('X-Forwarded-Host', url.hostname); // پنل این را به‌عنوان آدرس/SNI/Host ذخیره می‌کند
    headers.set('X-Forwarded-Proto', url.protocol.replace(':', ''));

    const init = { method: request.method, headers, redirect: 'manual' };
    if (request.method !== 'GET' && request.method !== 'HEAD') init.body = await request.arrayBuffer();

    const res = await fetch(target, init);
    const out = new Headers(res.headers);
    out.delete('content-encoding'); // بگذار Cloudflare فشرده‌سازی را انجام دهد
    out.delete('content-length');
    return new Response(res.body, { status: res.status, headers: out });
  }
};
```

نکته: کوکی ورود (`bk_session`) روی دامنهٔ Worker ست می‌شود، پس ورود و تب **Usage** از همان آدرس Worker کار می‌کنند. دامنهٔ Worker باید خودش در ایران فیلتر نباشد (دامنهٔ شخصی بهترین گزینه است).

### بهینه‌ساز کانفیگ
دکمهٔ **بهینه‌ساز کانفیگ** بالای پنل، داخل منو و کنار کانفیگ‌ها است. لینک را باز می‌کند (تب جدید): https://arastey.github.io/cf-optimizor/ — کانفیگ را آنجا بچسبان.

### مصرف (Usage)
تب **Usage** مصرف هر کانفیگ را نشان می‌دهد: آپلود و دانلود، امروز، این ماه (از ۰۰:۰۰ UTC روز اول) و کل، به‌اضافهٔ سرعت لحظه‌ای. این عدد تخمینی است؛ عدد واقعی **Account → Usage** در Replit است. شمارنده در پوشهٔ داده می‌ماند و با هر **Publish** صفر می‌شود. **Sync** عدد ماه را با Replit یکی می‌کند و **Reset counter** همه‌چیز را پاک می‌کند (فقط بعد از ورود). Secret اختیاری `TRAFFIC_LIMIT_GB` نوار پیشرفت و هشدار ۸۰٪ و ۱۰۰٪ را روشن می‌کند. روی موبایل جدول به یک کارت برای هر کانفیگ تبدیل می‌شود.

### بیدار نگه‌داشتن (Autoscale)
خود برنامه حدود هر ۴ دقیقه یک درخواست کوچک به خودش روی همان دستگاه می‌زند تا دیپلوی Autoscale دیرتر بخوابد. این درخواست جزو ترافیک Usage حساب نمی‌شود و هیچ رمزی را لاگ نمی‌کند. این کار خاموش‌شدن اپ رایگان بعد از حدود ۳۰ روز را برنمی‌دارد.

### رمز و کانفیگ بعد از Publish
فایل‌های اپ منتشرشده با هر Publish پاک می‌شوند. برای ثابت ماندن، در **Publishing → Secrets** (همه اختیاری) بگذار: `UUID`، `PANEL_PASSWORD`، `SUB_TOKEN`.

## English

### Setup
1. Replit: **Create App → Import from GitHub** → this repo.
2. **Publish** (Reserved VM or Autoscale). You get `https://<name>.replit.app`.
3. Open `https://<name>.replit.app/knight` and create the panel password.
4. Copy the configs or the subscription. Every link is port **443** with **TLS**.

Port, domain and storage are detected automatically. No variable is required.

### If `replit.app` is filtered
Put a **Cloudflare Worker** in front of `<name>.replit.app` and set:

`headers.set("X-Forwarded-Host", new URL(request.url).hostname)`

Open the panel through the Worker (`https://<worker>.workers.dev/knight`). The panel saves that hostname and uses it as the address, SNI and Host in every config and in the subscription, so nobody has to edit a link by hand. A later visit through `replit.app` does not replace it. A `DOMAIN` Secret, if you set one, still wins.

Complete Worker (do not rewrite paths — the panel, `app.css`, `app.js`, `/health` and the Usage API all have to pass through):

```js
export default {
  async fetch(request) {
    const url = new URL(request.url);
    const ORIGIN = 'https://<name>.replit.app'; // your published Replit URL

    const target = new URL(url.pathname + url.search, ORIGIN);
    const headers = new Headers(request.headers);
    headers.delete('host');
    headers.set('X-Forwarded-Host', url.hostname); // the panel saves this as address / SNI / Host
    headers.set('X-Forwarded-Proto', url.protocol.replace(':', ''));

    const init = { method: request.method, headers, redirect: 'manual' };
    if (request.method !== 'GET' && request.method !== 'HEAD') init.body = await request.arrayBuffer();

    const res = await fetch(target, init);
    const out = new Headers(res.headers);
    out.delete('content-encoding'); // let Cloudflare do the compression
    out.delete('content-length');
    return new Response(res.body, { status: res.status, headers: out });
  }
};
```

The login cookie (`bk_session`) is set on the Worker's domain, so signing in and the **Usage** tab work from the Worker address too. The Worker's own hostname must not be filtered in Iran (a custom domain is the safest choice).

### Config optimizer
The **بهینه‌ساز کانفیگ** button (header, menu, and next to the configs) opens https://arastey.github.io/cf-optimizor/ in a new tab. Paste a config there.

### Usage
The **Usage** tab shows upload and download per config for today, this month (from 00:00 UTC on the 1st) and all time, plus live speed. It is an estimate; Replit's **Account → Usage** page is the real meter. The counter is saved in the data folder and resets on every **Publish**. **Sync** matches this month to Replit's number. **Reset counter** clears it (login required). Optional Secret `TRAFFIC_LIMIT_GB` adds a progress bar and warnings at 80% and 100%. On phones the table turns into one card per config.

### Keep-awake (Autoscale)
The app calls itself on localhost about every 4 minutes so an Autoscale deployment is less likely to sleep. That request is not counted in Usage and it logs no secrets. It does not stop Replit from shutting a free app down after about 30 days.

### Password and configs after Publish
A published app's files are wiped on every Publish. To keep them, add these optional **Publishing → Secrets**: `UUID`, `PANEL_PASSWORD`, `SUB_TOKEN`.

<details>
<summary><b>Optional settings (not required) / تنظیمات اختیاری</b></summary>

| Secret | Default | Meaning |
|---|---|---|
| `DOMAIN` | Replit domain, or the Worker host you opened the panel with | Force the domain in links |
| `UUID` | random, saved until the next Publish | Fixed UUID |
| `PANEL_PASSWORD` | – | Fixed panel password (8+ characters) |
| `SUB_TOKEN` | random | Fixed subscription token |
| `SETUP_KEY` | empty | Extra key the first time the password is created |
| `RESET_PANEL_PASSWORD` | – | `true`, publish, set a new password, then delete it |
| `FORCE_HOST` | – | Force the link address; SNI and Host stay on the domain |
| `LINK_FP` | `chrome` | uTLS fingerprint, or `none` |
| `LINK_ALPN` | `http/1.1` | ALPN for WebSocket links |
| `LINK_PORT` / `LINK_TLS` | `443` / on | Not needed on Replit |
| `ENABLE_XHTTP` and the other `ENABLE_*` flags | on | Set `false` to hide one config |
| `TRAFFIC_LIMIT_GB` | none | Monthly allowance (GiB) for the Usage bar |
| `BK_DATA_DIR` | `./bk-data` | Data folder |
| `PORT` | `8080` | Listening port (`.replit` maps 8080 → 80) |

</details>
