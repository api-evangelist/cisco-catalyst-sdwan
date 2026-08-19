---
name: cisco-catalyst-sdwan-tunnel-troubleshooting
description: Diagnose a degraded or down SD-WAN tunnel between two sites using Cisco Catalyst SD-WAN Manager real-time monitoring — BFD, control connections, TLOCs, application-aware routing SLA and interface state.
api: Cisco Catalyst SD-WAN Manager API
generated: '2026-08-19'
method: generated
source: openapi/cisco-catalyst-sdwan-monitoring-and-troubleshooting-openapi.json
operations:
  - createConnectionsSummary
  - createRealTimeConnectionList_RealTimeMonitoringDeviceControl_2703
  - createBFDSummary
  - createBFDSessions
  - createTLOCSummary
  - getDeviceTlocStatus
  - createAppRouteStatisticsList
  - createAppRouteSlaClassList
  - getDeviceInterface
  - createOMPSummary
  - createOMPSessionList
  - getRawAlarmData
---

# Tunnel troubleshooting

Work a degraded or down site-to-site tunnel from the overlay inward. Every operation here is a read
and takes a `deviceId` query parameter (the device `system-ip`) to scope it to one device.

## 0. Authenticate

See `cisco-catalyst-sdwan-fabric-inventory`. You need `Authorization: Bearer <jwt>` plus
`X-XSRF-TOKEN`. These calls need `Device Monitoring-read`.

## 1. Is the control plane up?

`createConnectionsSummary` — `GET /device/control/summary?deviceId=<system-ip>`

`createRealTimeConnectionList_RealTimeMonitoringDeviceControl_2703` — `GET /device/control/connections?deviceId=<system-ip>`

If a device has no control connection to a vSmart, it will never form data-plane tunnels. Stop here
and fix control before looking at BFD.

## 2. Is BFD up?

`createBFDSummary` — `GET /device/bfd/summary?deviceId=<system-ip>`

`createBFDSessions` — `GET /device/bfd/sessions?deviceId=<system-ip>`

BFD sessions are the data-plane tunnels. `sessions` gives you per-tunnel state, the remote system-ip,
the local and remote colors, and the transition count. A high transition count on an otherwise "up"
session is a flapping transport, not a healthy one.

## 3. Which TLOC is failing?

`createTLOCSummary` — `GET /device/bfd/tloc?deviceId=<system-ip>`

`getDeviceTlocStatus` — `GET /device/tloc?deviceId=<system-ip>`

A TLOC is a (system-ip, color, encapsulation) tuple. Narrowing to the failing TLOC tells you which
physical circuit is at fault.

## 4. Is the transport interface itself healthy?

`getDeviceInterface` — `GET /device/interface?deviceId=<system-ip>`

Check admin/oper state, errors and drops on the interface behind the failing color.

## 5. Is routing converged?

`createOMPSummary` — `GET /device/omp/summary?deviceId=<system-ip>`

`createOMPSessionList` — `GET /device/omp/peers?deviceId=<system-ip>`

For the routes themselves use `createAdvertisedRoutesList` (`GET /device/omp/routes/advertised`) and
`createReceivedRoutesList` (`GET /device/omp/routes/received`). There is **no** `/device/omp/routes`
endpoint in the published contract, despite what some tooling calls.

## 6. Is the application actually meeting SLA?

`createAppRouteStatisticsList` — `GET /device/app-route/statistics?deviceId=<system-ip>`

`createAppRouteSlaClassList` — `GET /device/app-route/sla-class?deviceId=<system-ip>`

Loss, latency and jitter per tunnel against the configured SLA classes. A tunnel can be up and still
be failing the SLA that matters to the application.

## 7. Correlate with alarms

`getRawAlarmData` — `GET /alarms`

Filter to the affected devices and the time window. `getAlarmSeverityCustomHistogram`
(`GET /alarms/severity/summary`) gives the shape of the incident quickly.

## Cautions

- These are **real-time** endpoints: they reach out to the device on every call. Cisco documents them
  for troubleshooting only. Do not build a polling monitor on them.
- Bulk statistics (`/data/device/statistics/`) allows only **2 concurrent requests**, and bulk APIs
  overall are capped at 48 requests/minute.
- Responses have no error schema. On `403` check the caller's role, not the resource.
