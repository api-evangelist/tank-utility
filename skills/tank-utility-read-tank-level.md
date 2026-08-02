---
name: Read propane tank level
description: Authenticate to Tank Utility, list the monitors on the account, and read each tank's current fuel level, temperature, and last report time.
api: openapi/tank-utility-devices-openapi.yml
operations: [getToken, getDevices, getDevice]
---

# Read propane tank level (Tank Utility)

Use the Tank Utility API to check how much propane is left in each monitored
tank. The API is read-only; base URL is `https://data.tankutility.com/api`.

## Steps

1. **Get a token** — call `getToken` (`GET /getToken`) with HTTP Basic auth,
   using the account email as the username and the account password. On success
   you get `{ "token": "..." }`. The token is valid for about 24 hours — cache
   it and refresh only when a device call returns `401`.

2. **List devices** — call `getDevices` (`GET /devices?token=<token>`). The
   response is `{ "devices": ["<deviceId>", ...] }` — the device IDs on the
   account.

3. **Read each device** — for every ID, call `getDevice`
   (`GET /devices/{deviceId}?token=<token>`). The response contains a `device`
   object whose `lastReading` holds `tank` (fuel level as a percentage of
   capacity), `temperature`, and `time`/`time_iso` (when it was reported).

## Rules

- **Auth model**: Basic auth is used *only* on `getToken`. Every other call
  authenticates with the `token` query parameter — never resend Basic auth.
- **Token expiry**: on `401 Unauthorized`, re-run `getToken` for a fresh token
  and retry once. (See `conventions/tank-utility-conventions.yml`.)
- **Errors**: the envelope is `{ statusCode, message }`, not RFC 9457. A
  `getToken` `400 EMAIL_NOT_FOUND` means the email is not a registered account;
  a device `404` means that ID is not on this account. (See
  `errors/tank-utility-error-codes.yml`.)
- **Read-only**: all operations are GET; there are no writes and no webhooks —
  poll `getDevice` on a schedule if you need updates.
