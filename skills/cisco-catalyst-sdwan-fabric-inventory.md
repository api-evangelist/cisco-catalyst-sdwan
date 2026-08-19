---
name: cisco-catalyst-sdwan-fabric-inventory
description: Authenticate to a Cisco Catalyst SD-WAN Manager instance and take a full inventory of the fabric — every device, which are reachable, control-plane health, and certificate status.
api: Cisco Catalyst SD-WAN Manager API
generated: '2026-08-19'
method: generated
source: openapi/cisco-catalyst-sdwan-monitoring-and-troubleshooting-openapi.json, openapi/cisco-catalyst-sdwan-others-openapi.json, https://developer.cisco.com/docs/sdwan/authentication/
operations:
  - listAllDevices
  - listReachableDevices
  - getDevicesDetails
  - createConnectionsSummary
  - createDeviceSystemStatusList
  - getCertificateStats
  - getVedgeInventory
---

# Fabric inventory

Take a complete, read-only picture of one Cisco Catalyst SD-WAN fabric.

## Before you start

- The API is served by **the customer's own SD-WAN Manager controller**, not by Cisco.
  The base URL is `https://<sdwan-manager-host>:8443/dataservice`. There is no shared host.
- Every request must be HTTPS.
- All the operations below are reads. Each still requires the caller's user group to hold the
  operation's `x-roles-required` permission — mostly `Device Monitoring-read` and `Certificates-read`.

## 1. Authenticate

On SD-WAN Manager 20.18.1 and later, use JWT:

```
POST /jwt/login
Content-Type: application/json
{"username": "...", "password": "...", "duration": 1800}
```

The response body carries `token` (the access token) and `csrf` (the XSRF token). Send both on
every subsequent call:

```
Authorization: Bearer <token>
X-XSRF-TOKEN: <csrf>
```

On older releases, POST to `/j_security_check` for a `JSESSIONID` cookie, then
`GET /dataservice/client/token` for the XSRF token.

**None of these three login endpoints appear in the published OpenAPI.** They are documented in prose
only, so do not expect to find an operationId for them.

## 2. List every device

`listAllDevices` — `GET /device`

Returns the whole fabric: WAN edges plus the vManage, vSmart and vBond controllers. Accepts an
optional `model` filter. This is the anchor call; every later step keys off `uuid` or `system-ip`.

## 3. Split reachable from unreachable

`listReachableDevices` — `GET /device/reachable`

Diff this against step 2. Anything present in step 2 and missing here is the first thing to look at.

## 4. Get the control-plane summary

`createConnectionsSummary` — `GET /device/control/summary`

Control connections are the fabric's nervous system. A device that is "reachable" but has no control
connections is not actually participating in the overlay.

## 5. Get per-device system status

`createDeviceSystemStatusList` — `GET /device/system/status`

CPU, memory and uptime per device. Pass `deviceId` to scope it to one device.

## 6. Check certificate health

`getCertificateStats` — `GET /certificate/stats/summary`

Expiring or invalid certificates take devices out of the overlay with no warning. Check this on every
inventory pass, not only when something is broken.

## 7. Inventory the edge hardware

`getVedgeInventory` — `GET /device/vedgeinventory/detail`

Chassis, serial and validity state for the WAN edge inventory.

`getDevicesDetails` — `GET /system/device/{deviceCategory}` — gives the configuration-database view
of the same devices, which can differ from the live view when a device is offline.

## Conventions that will bite you

- **Pagination differs by data source.** Configuration data uses `offset`/`limit`. Statistics data
  uses `scrollId`/`count`. Device data paginates however the device decides.
- **No consistency guarantee.** Every response is a snapshot at invocation time. Two calls in the
  same loop can disagree; do not treat them as one transaction.
- **Rate limits.** 100 requests/second for general APIs, 48 requests/minute for bulk APIs, 2
  concurrent requests on `/data/device/statistics/`, 250 concurrent sessions. Exhaustion returns
  `429` — with **no** `Retry-After` header and no rate-limit headers at all, so back off on your own
  schedule.
- **Real-time monitoring calls are CPU intensive.** Cisco documents them for troubleshooting only.
  Do not poll them continuously.
- **No idempotency.** There is no idempotency key anywhere in this API. Retrying is only safe on the
  reads above.

## Errors

`403` means the caller's role does not carry the operation's `x-roles-required` permission — not that
the resource is missing. `400`, `403` and `500` are declared on essentially every operation and carry
**no response schema**, so you get a status code and a one-line description, nothing parseable.
