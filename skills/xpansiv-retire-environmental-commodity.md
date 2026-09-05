---
name: xpansiv-retire-environmental-commodity
description: Permanently retire an environmental commodity instrument through Xpansiv Connect, with the pre-flight and confirmation steps this API requires because retirement is irreversible and unprotected by idempotency.
api: Xpansiv Connect API
base_url: https://connect.xpansiv.com/app/api/v1
generated: '2026-09-04'
method: generated
source: openapi/xpansiv-connect-openapi.yml + conventions/xpansiv-conventions.yml + https://developer.xpansiv.com/developer-portal/xpansiv-connect/getting-started
operations:
  - getReferenceData
  - accountSearch
  - searchAccountPositions
  - getRegistryRules
  - createRetirement
  - checkStatus
  - searchRetirements
---

# Retire an environmental commodity instrument (Xpansiv Connect)

**Read this first.** `createRetirement` is terminal. No cancel, void, reverse, unretire or
restore operation exists for a retirement in any Xpansiv API — retirement is how a
certificate is permanently withdrawn from circulation so its environmental attribute can
be claimed exactly once. There is also **no idempotency key** on this or any other
Xpansiv write: a timeout followed by a blind retry can retire twice. Every step below
exists to make those two facts survivable.

## 1. Get a token

Bearer tokens come from the Xpansiv authorization server, not from the API host.

```
POST https://auth.xpansiv.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=http://auth0.com/oauth/grant-type/password-realm
realm=Username-Password-Authentication
audience=https://xpansiv/platform
client_id={CLIENT_ID}&client_secret={CLIENT_SECRET}
username={USERNAME}&password={PASSWORD}
```

Returns `access_token` with `expires_in` 86400. Use `Authorization: Bearer {access_token}`
on every call below. Credentials are issued by Xpansiv on request; there is no self-serve
signup. Work against `https://uat.preprod.connect.xpansiv.com/app/api/v1` with
`https://auth.preprod.xpansiv.com/oauth/token` until the flow is proven.

## 2. Resolve reference data and the account

- `getReferenceData` — `GET /reference-data`. Returns the program, registry and status
  code lists (`Ref.Program`, `Ref.CarbonProgram`, `Ref.RecProgram`, `Ref.AccountStatus`).
  Never hardcode a `ProgramCode` or `RetirementProgramCode`; read it from here.
- `accountSearch` — `GET /account/{AccountIdentifier}`. Confirm the account exists and is
  in a usable status before doing anything else.

## 3. Confirm you actually hold what you intend to retire

`searchAccountPositions` — `POST /portfolio/account/{AccountIdentifier}/position/action/search`.
Xpansiv models list retrieval as POST-with-criteria, so paging and filters go in the
request body, not the query string. Match the position you intend to retire by program and
instrument before proceeding. Retiring a position you do not hold fails with 422, but
retiring the *wrong* position succeeds and cannot be undone.

## 4. Read the program's rules — this is the only pre-flight the API offers

`getRegistryRules` — `GET /retirements/program/{RetirementProgramCode}/rules`.

Each registry program imposes its own constraints on what may be retired, on whose behalf,
and with what beneficiary and reason metadata. 422 Unprocessable Entity is the dominant
failure on this endpoint family and it is almost always a rules violation. Fetch the rules
and validate your payload against them locally. There is no dry-run or simulate mode.

## 5. Retire

`createRetirement` — `POST /retirements/account/{AccountIdentifier}/program/{RetirementProgramCode}/action/create`.

Before sending:

- Record your own client-side transaction id. Xpansiv Connect declares no `correlationId`
  (the registry and Optimal APIs do; Connect does not), so the only duplicate protection
  is the one you build.
- Do not wrap this call in a generic retry-on-5xx or retry-on-429 policy. See step 6.

## 6. Confirm — never retry blind

`checkStatus` — `POST /retirements/account/{AccountIdentifier}/action/check-status`.

If step 5 times out, returns 5xx, or returns 429, **do not resend it**. Call `checkStatus`,
and if that is inconclusive call `searchRetirements`
(`POST /retirements/account/{AccountIdentifier}/action/search`) and look for the retirement
you just attempted. Only resubmit once you have positively established that nothing landed.

429 carries no `Retry-After` and no `RateLimit-*` header on this API, so there is no
published backoff interval — use exponential backoff and re-confirm state on each attempt.

## Error handling

| Status | Meaning here | Do |
|---|---|---|
| 401 | Token expired (24h) or missing | Re-run step 1 |
| 403 | Token valid, account lacks rights on this program | Stop; this is a permissions issue, not a payload issue |
| 422 | Program rules violation | Re-read `getRegistryRules`; fix the payload; do not retry unchanged |
| 429 | Rate limited | Backoff, then confirm with `checkStatus` before resending |
| 5xx | Unknown outcome | Confirm with `checkStatus` before resending — never assume it failed |

Error bodies are plain `application/json`; Xpansiv publishes no RFC 9457 problem type and
no shared error envelope. Branch on status code, not on a body field.
