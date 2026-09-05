# Xpansiv

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Xpansiv is market infrastructure for global environmental and energy commodities — carbon
credits, renewable energy certificates, water, low-carbon fuels and recycled materials. It
operates CBL, the largest spot exchange for environmental commodities, the Evolution
Markets and OTX execution desks, the H2OX water market, and the environmental registries it
acquired with APX in 2022 (NAR, TIGR, I-REC, I-TRACK-G and the digital fuels registries).

## What this profile found

Xpansiv publishes a substantial, machine-readable API surface at
[developer.xpansiv.com](https://developer.xpansiv.com/) — a Redocly portal that also serves
an `llms.txt` and a live remote MCP server.

- **11 OpenAPI descriptions, 110 operations, 492 schemas** — harvested verbatim to
  `openapi/_original/`. Xpansiv Connect (28 ops), Xpansiv Managed Solutions (29), the
  Optimal Outcomes suite (32 across six services), the NAR (9) and TIGRS (5) registry client
  APIs, and the APX Power Markets file registry (7).
- **A FIX 4.4 marketplace protocol specification** — CBL order entry and real-time market
  data are FIX, not REST, and Xpansiv publishes its full rules of engagement.
- **A live remote MCP server** at `https://developer.xpansiv.com/mcp`, answering
  `tools/list` anonymously with 7 tools. It is a *documentation* server: none of its tools
  binds to any of the 110 business operations (see `mcp/xpansiv-tool-crosswalk.yml`).
- **One first-party SDK** — `xpansiv-data` on PyPI, v1.0.2.post1, published 2026-03-12.
- **Four separate credential systems** behind one `Authorization: Bearer` header, with no
  token accepted across families — a direct consequence of growth by acquisition.

Three gaps are worth naming because they carry real risk on a catalog that moves and cancels
financial instruments: **no idempotency key exists on any of the 110 operations**,
**retirements are irreversible** with no cancel or restore path anywhere, and **no rate
limit is published** despite 429 being declared on 15 operations. The `conventions/`,
`errors/` and `rate-limits/` artifacts record each with evidence.

Xpansiv publishes no status page, no changelog, no security.txt, no trust center and no
public pricing.

## Artifacts

`openapi/` · `overlays/` · `mcp/` · `llms/` · `well-known/` · `packages/` ·
`authentication/` · `scopes/` · `conventions/` · `errors/` · `conformance/` · `lifecycle/` ·
`data-model/` · `rate-limits/` · `plans/` · `sandbox/` · `security/` · `skills/`

- Website: https://www.xpansiv.com/
- Developer portal: https://developer.xpansiv.com/
- Support: https://support.xpansiv.com/
