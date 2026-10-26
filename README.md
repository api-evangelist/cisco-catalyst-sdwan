# Cisco Catalyst SD-WAN

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

Cisco Catalyst SD-WAN, built on the Viptela platform Cisco acquired in 2017, is Cisco's wide-area network
overlay: centralized policy, application-aware routing, and secure transport across MPLS, broadband and LTE.
Its controller — SD-WAN Manager, formerly vManage — exposes a REST API of **4,138 published operations across
2,841 paths**, documented on Cisco DevNet as **thirteen OpenAPI 3.1.0 documents** at release 26.1.0.

## Ownership

Part of the Cisco family (acquired 2017).

## Contract status — corrected 2026-08-19

An earlier round of this profile recorded *"the contract is served by each customer's own on-premises
controller, so there is no central anonymously fetchable specification."* **That was wrong, and it is
corrected here.**

The API **host** is customer-operated — every SD-WAN Manager instance belongs to a customer, and the base URL
is `https://<sdwan-manager-host>:8443/dataservice`. But the **contract** is not customer-held. Cisco serves
the complete specification anonymously from its DevNet documentation CDN
(`pubhub.devnetcloud.com/media/cisco-catalyst-sd-wan-26-1-api-guide/`) as one self-contained OpenAPI 3.1.0
fragment per operation. All 4,138 were fetched (zero errors) and reassembled verbatim into the thirteen
documents in `openapi/`.

Ownership was verified from what the specification says about itself, not from where it was fetched:
`info.contact.email` is `vmanage@cisco.com`, `info.license.url` points at Cisco's own SD-WAN solutions page,
and `info.version` is `26.1.0+2026-01-06`.

## What is in the contract

| Specification | Operations | Paths |
|---|---|---|
| Monitoring and Troubleshooting | 1,068 | 917 |
| UX 1.0 Configuration | 862 | 562 |
| Others (inventory, certificates, software, multitenancy, licensing) | 718 | 602 |
| Feature Profiles - SD-Routing | 429 | 175 |
| Feature Profiles - SD-WAN Transport | 224 | 108 |
| SD-WAN Services (multicloud, interconnect) | 180 | 134 |
| Feature Profiles - SD-WAN Service | 160 | 76 |
| Feature Profiles - Mobility and NFV | 137 | 64 |
| Feature Profiles - Others | 113 | 52 |
| Partner Integrations | 76 | 62 |
| Feature Profiles - SD-WAN System | 73 | 35 |
| Administration and Settings | 59 | 34 |
| UX 2.0 Configuration | 39 | 20 |

Every operation carries a unique `operationId` and a tag; 3,626 carry an in-spec example; **240 are marked
deprecated**. Every operation also carries an `x-roles-required` extension naming the SD-WAN Manager RBAC
permission it needs — 515 distinct role expressions in all, captured in `scopes/`.

## What the contract does not carry

Recorded honestly, because an agent needs to know:

- **No `securitySchemes`.** The authentication model — JWT bearer from `POST /jwt/login`, the legacy
  `JSESSIONID` session, and the `X-XSRF-TOKEN` header on writes — exists only in the DevNet prose. None of the
  three login endpoints appears in any of the 4,138 operations. `overlays/` adds what the docs define.
- **A relative server.** `servers[]` is `/dataservice`; the host is a variable by design.
- **No error schema.** `400`, `403` and `500` are declared on nearly every operation with a description and no
  body schema. `401`, `429` and `503` are documented in prose and declared on zero operations.
- **No idempotency.** No key, no replay semantics, anywhere.
- **No rate-limit headers.** Limits are published as static numbers (100 req/s general, 48 req/min bulk, 2
  concurrent on bulk statistics, 250 concurrent sessions); exhaustion returns `429` with no `Retry-After`.

## Agent surface

Two MCP servers are published in the CiscoDevNet GitHub organization —
`catalyst-sdwan-mcp-community` (45 tools) and `wan-automation-examples/catalystwan/mcp-sdwan` (6 tools).
**Both are stdio-only**: neither is a hosted endpoint an agent can reach, and neither is on npm or PyPI. That
is a property of the product, not an oversight — there is no shared controller to host against.

`mcp/cisco-catalyst-sdwan-tool-crosswalk.yml` binds each tool to the operation it actually calls, read from
the server source. Thirty-five of 45 tools bind to a published operation. **Ten call vManage paths that appear
in no published specification** — `/system/device/controllers`, `/device/omp/routes`, `/device/vedgeinventory`,
`/clusterManagement/clusterStatus`, `/template/policy`, `/template/policy/definition` and others.

No A2A agent card is served on any Cisco host probed.

## Ecosystem currency

| Package | Version | Last published |
|---|---|---|
| `catalystwan` (PyPI, Python SDK) | 0.41.6 | 2026-07-31 |
| `cisco-sdwan` (PyPI, Sastre CLI) | 1.28 | 2026-06-05 |
| `CiscoDevNet/sdwan` (Terraform) | 0.11.4 | 2026-07-28 |
| `cisco.sdwan` (Ansible Galaxy) | 0.3.5 | 2024-11-29 |
| `cisco.catalystwan` (Ansible Galaxy) | 0.3.2 | 2024-11-29 |
| `cisco.sdwan_deployment` (Ansible Galaxy) | 0.3.3 | 2024-11-28 |

The three Ansible collections have not shipped since November 2024, while the API they wrap has moved 20.18 →
26.1 with 32 operations deleted and 76 deprecated. `cisco-open/cisco-catalyst-wan-sdk`, the SDK's former home,
was archived 2024-11-21; development continues at `cisco-en-programmability/catalystwan-sdk`.

## Verified links

Probed 2026-08-19 with the status shown.

- [Developer portal / documentation / API reference](https://developer.cisco.com/docs/sdwan/) — 200
- [Getting started](https://developer.cisco.com/docs/sdwan/getting-started/) — 200
- [Authentication](https://developer.cisco.com/docs/sdwan/authentication/) — 200
- [Versioning and deprecation](https://developer.cisco.com/docs/sdwan/versioning-and-deprecation/) — 200
- [API changelog](https://developer.cisco.com/docs/sdwan/api-changelog/) — 200
- [Rate limits and pagination](https://developer.cisco.com/docs/sdwan/browsing-returned-results-sorting-results-filtering-results-and-rate-limits/) — 200
- [Errors and troubleshooting](https://developer.cisco.com/docs/sdwan/errors-and-troubleshooting/) — 200
- [Developer support](https://developer.cisco.com/docs/sdwan/developer-support/) — 200
- [Reservable DevNet sandbox](https://devnetsandbox.cisco.com/RM/Diagram/Index/ed2c839d-621e-4c55-b176-db2457baf4c8?diagramType=Topology) — 200
- [SD-WAN programmability learning track](https://developer.cisco.com/learning/tracks/sd-wan_programmability/) — 200
- [Terraform provider](https://registry.terraform.io/providers/CiscoDevNet/sdwan/latest) — 200
- [CiscoDevNet on GitHub](https://github.com/CiscoDevNet) — 200
- [Cisco security vulnerability policy (PSIRT)](https://sec.cloudapps.cisco.com/security/center/resources/security_vulnerability_policy.html) — 200
- [Cisco Trust Portal](https://trustportal.cisco.com/c/r/ctp/trust-portal.html) — 200
- [Cisco blog, SD-WAN](https://blogs.cisco.com/tag/sd-wan) — 200
- [Parent company profile](https://apis.io/providers/cisco/)

`www.cisco.com` returned **403** (edge bot protection) to every probe in this run, including
`/.well-known/security.txt`. That is recorded as a blocked probe, not as an absence — Cisco does serve an
RFC 9116 `security.txt` there, captured in the parent Cisco profile. The terms-of-service and privacy-policy
links in `apis.yml` are Cisco's canonical legal pages, carried over from that parent profile rather than
re-probed here.
