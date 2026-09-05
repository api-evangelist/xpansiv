---
name: xpansiv-submit-iso-schedule
description: Submit an ISO scheduling file to APX Power Markets and retrieve its asynchronous validation result — the step most integrations miss, because a successful upload does not mean an accepted schedule.
api: APX Power Markets File Registry API
base_url: https://pm-file-api.apx.com
generated: '2026-09-04'
method: generated
source: openapi/xpansiv-apx-power-markets-file-registry-openapi.yml + https://developer.xpansiv.com/developer-portal/xpansiv-power/rest_api
operations:
  - uploadFile
  - getStatus
  - getFile
  - getList
  - getList_1
  - getScheduleData
  - getReport
---

# Submit an ISO schedule and read its validation result (APX Power Markets)

Covers CAISO, ERCOT, PJM, SPP, MISO, ISONE and NYISO scheduling through the APX MarketSuite
file registry.

## 1. Token

This family uses a different authorization server from Xpansiv Connect — RFC 6749 password
grant, not the Auth0 password-realm extension:

```
POST https://apxjwtauthprod.apx.com/oauth/token      (UAT: https://apxjwtauthuat.apx.com/oauth/token)
Authorization: Basic {base64(clientId:clientSecret)}
Content-Type: application/x-www-form-urlencoded

Username={service user}&Password={service password}&grant_type=password
```

Returns a short-lived JWT with `scope: access` — the single, non-subdividable scope this
server issues. Send it as `Authorization: Bearer {access_token}`.

## 2. Build the payload against the XSD, not against the OpenAPI

The OpenAPI describes the *envelope*. The scheduling content itself is governed by the
**APXScheduleAPI XSD** (`targetNamespace http://service.apx.com/schedule`), published at
<https://developer.xpansiv.com/developer-portal/xpansiv-power/rest_api/xsds>. Validate
locally against that schema before uploading — its inline change history is the only dated
release record Xpansiv publishes for this API (most recently "May 6, 2026 Added 4 new
products for CAISO DAME market change"), so re-check it when a market change lands.

Region matters: per the XSD history, MISO and PJM use the `ScheduleData` schema rather than
the `ParameterSet` `Region` enumeration.

## 3. Upload

`uploadFile` — `POST /fileRegistry/file`. Keep the returned **`fileHandle`**. It is the only
handle you get, and step 4 needs it. 413 means the payload exceeded the accepted size.

## 4. Read the validation result — the step that matters

`getStatus` — `GET /fileRegistry/file/{fileHandle}/status`.

**A 200 on `uploadFile` means the file was accepted for processing, not that the schedule
was accepted.** Validation runs asynchronously and its errors are only visible here. An
integration that checks the upload status code and stops will silently submit invalid
schedules. Poll `getStatus` until the file reaches a terminal state and read the validation
errors it returns.

## 5. Retrieve

- `getList` — `PUT /fileRegistry/listFiles` — list downloadable files.
- `getList_1` — `PUT /fileRegistry/listFilesSince` — incremental list; use this for polling
  rather than re-listing everything.
- `getScheduleData` — `PUT /fileRegistry/getScheduleData/{fileName}` — query ISO scheduling
  data.
- `getFile` — `GET /fileRegistry/file/{fileHandle}` — download a specific file.
- `getReport` — `PUT /reporting/getReport` — run a MarketSuite report and receive results.

Note the unusual verb choice: the query operations are `PUT`, not `GET` or `POST`. This is
what the published contract declares.

## Getting started faster

Xpansiv publishes a Postman collection and a sandbox environment for this API at
<https://developer.xpansiv.com/developer-portal/xpansiv-power/rest_api/postman>. The
environment ships variable names, not credentials — add your own.

## Auth is per-family

Credentials for this API are **not** interchangeable with Xpansiv Connect, Xpansiv Data or
Managed Solutions. Each product family has its own issuer and its own account.
