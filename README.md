# RSI Sniper V9 Compatible Dhan Backend

This backend is adapted to the uploaded RSI Sniper V9 Incremental PWA source. It retains the data-only architecture: strategy and entry signals remain in the PWA.

## Render settings
- Runtime: Node
- Build command: `npm install`
- Start command: `npm start`
- Health check path: `/api/health`

## Required Render environment variables
Set `DHAN_CLIENT_ID` and a current `DHAN_ACCESS_TOKEN` from Dhan. The backend does not generate or refresh Dhan access tokens from PIN/TOTP.

Set the VAPID variables from `ENVIRONMENT_VARIABLES.txt`. Keep `VAPID_PRIVATE_KEY` secret and do not place it in PWA/frontend files.

## V9 compatibility adjustments
- `/api/candles` accepts both `tf` and `timeframe`.
- Added `/api/history?segment=IDX_I&securityId=13` and compatible response fields `data` and `candles`.
- `/api/subscribe` accepts both lower-case and Dhan-style `ExchangeSegment` / `SecurityId` field names.
- Option-chain endpoint accepts `symbol`, `index`, or a configured `securityId` alias.

## Useful endpoints
- `GET /api/health`
- `GET /api/status`
- `GET /api/state`
- `GET /api/spot?symbol=NIFTY`
- `GET /api/candles?symbol=NIFTY&tf=1m&limit=500`
- `GET /api/history?segment=IDX_I&securityId=13`
- `GET /api/option-chain?symbol=NIFTY`
- `POST /api/subscribe`
- `GET /api/notifications/vapid-public-key`
- `POST /api/push/subscribe`
- `POST /api/push/test`
- WebSocket: `/ws`

## Important
Syntax validation has been run on `server.js`; live Dhan authentication, market-hours feed, and end-to-end PWA notification delivery still require deployment with valid Dhan credentials and browser permission. The option-chain and REST endpoints remain subject to Dhan's entitlements and rate limits. `CORS_ORIGIN=*` is for initial testing only; restrict it for public deployment.
