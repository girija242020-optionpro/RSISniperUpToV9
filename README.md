# Dhan Universal Market Data Backend

Personal data-provider backend for RSI Sniper PWA V7/V8/V9. It does not implement trading strategy or place orders; each PWA should make its own indicator/entry/risk decisions.

## Render setup
- Root Directory: `.` (blank if repository root is the project root)
- Build Command: `npm install`
- Start Command: `npm start`
- Runtime: Node 20+
- Health check path (optional): `/api/health`

Upload the contents of this folder to the root of a GitHub repository. Set Render environment variables from `RENDER_ENV_COPY.txt`. Set `DHAN_CLIENT_ID`, `DHAN_PIN`, and `DHAN_TOTP_SECRET` privately in Render. With TOTP enabled, the backend requests a fresh 24-hour access token when it starts/reconnects. Do not commit `.env` or credentials.

## Dashboard
Open `https://YOUR-RENDER-SERVICE.onrender.com/` or `/dashboard`. It refreshes status every 5 seconds and shows feed state, token state, last tick, reconnects, subscribed indices, stale instruments, and endpoint shortcuts.

## Data API
- `GET /health`, `/api/health`
- `GET /api/status`, `/api/state`, `/api/config`, `/api/analytics`
- `GET /api/spot?symbol=NIFTY` (also BANKNIFTY, FINNIFTY, MIDCPNIFTY, SENSEX)
- `GET /api/ticks?symbol=NIFTY`
- `GET /api/candles?symbol=NIFTY&limit=500`
- `GET /api/history?segment=IDX_I&securityId=13`
- `GET /api/depth?symbol=NIFTY`
- `POST /api/subscribe` with `{ "instruments": [{"exchangeSegment":"IDX_I","securityId":"13"}] }`
- `GET /api/option-chain?underlying=NIFTY&expiry=YYYY-MM-DD`
- `GET /api/push/vapid-public-key`
- `POST /api/push/subscribe` with `{ "subscription": <PushSubscription JSON> }`
- `POST /api/push/test`
- WebSocket: `wss://YOUR-RENDER-SERVICE.onrender.com/ws` (server sends `{type:"tick",data:...}` and status messages)

## Important limitations and operational notes
- The Dhan v2 binary packet layout is parsed for common ticker/quote/full fields. Exchange segment codes and depth field details should be verified against the current Dhan documentation and actual subscribed packet mode. The depth endpoint only reports fields the parser has actually decoded; it must not be treated as verified 20-level L20 data.
- Candles are assembled from received ticks and are not backfilled historical candles. `/api/history` only returns in-memory candles since this service started. A restart clears memory.
- Option chain REST call needs a valid expiry date and correct underlying security ID/segment; check actual Dhan response and account Data API entitlement. The backend proxies raw response; it does not invent option data.
- Render free instances can sleep/restart. For persistent, always-on market feed, use an always-on service plan and monitor `/api/status`.
- If `CLIENT_API_KEY` is set, protected endpoints require `X-API-Key`; keep this blank for the first dashboard test unless the frontend is updated to send it. Do not expose this personal endpoint publicly without suitable access control.
- `CORS_ORIGIN=*` is convenient for first smoke test but not recommended as final configuration. Set it to comma-separated HTTPS origins of your own PWAs once known.
- VAPID keys are one reusable pair for your own PWA family. Keep private key only in Render Environment. Push requires the PWA to register a service worker and send its PushSubscription to `/api/push/subscribe`. The push test endpoint verifies delivery only; the data backend does not decide RSI signals or automatically push trading alerts.

## Security
The Dhan PIN and TOTP secret were included in a chat message. Treat them as exposed: rotate/reset the PIN and regenerate the TOTP secret in Dhan, then set the new values only in Render. Never put them in the PWA, GitHub, ZIP source, screenshots, or public URLs.
