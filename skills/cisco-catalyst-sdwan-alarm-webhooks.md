---
name: cisco-catalyst-sdwan-alarm-webhooks
description: Register an outbound webhook so Cisco Catalyst SD-WAN Manager pushes alarms to your endpoint instead of you polling for them.
api: Cisco Catalyst SD-WAN Manager API
generated: '2026-08-19'
method: generated
source: openapi/cisco-catalyst-sdwan-monitoring-and-troubleshooting-openapi.json
operations:
  - createNotificationRule
  - updateNotificationRule
  - getNotificationRule
  - deleteNotificationRule
  - getRawAlarmData
  - getAlarmSeverityCustomHistogram
---

# Alarm webhooks

SD-WAN Manager has one push surface: **notification rules**. A rule binds an alarm name and severity
to an email list and/or an HTTP callback. Everything else in this API is polled.

## 0. Authenticate

`Authorization: Bearer <jwt>` plus `X-XSRF-TOKEN`. `createNotificationRule` requires
**`Settings-write`**.

## 1. Know which alarms exist

`getRawAlarmData` — `GET /alarms` — the live alarm feed; the `rule_name_display` / `name` fields tell
you the alarm identifiers in use in this fabric.

`getAlarmSeverityCustomHistogram` — `GET /alarms/severity/summary` — the severity distribution.

Pick the alarm name you want to be told about from what the fabric actually raises. There is no
separate published alarm-type registry in the contract.

## 2. Create the rule

`createNotificationRule` — `POST /notifications/rule`

The request body Cisco documents:

```json
{
  "notificationRuleName": "bfd-node-up",
  "alarmName": "BFD_Node_Up",
  "severity": "Medium",
  "webHookEnabled": true,
  "webhookUrl": "https://receiver.example.com/sdwan",
  "webhookUsername": "...",
  "webhookPassword": "...",
  "accountDetailsArray": ["ops@example.com"],
  "emailThreshold": 5,
  "devicesAttached": "<device uuid>",
  "sourceVpn": 512,
  "vpn": 10,
  "vpnIpSubnet": "192.168.1.0/24"
}
```

Returns `202 Accepted`.

## 3. Manage the rule

`getNotificationRule` — `GET /notifications/rules` — all rules, or one by id.

`updateNotificationRule` — `PUT /notifications/rule`

`deleteNotificationRule` — `DELETE /notifications/rules`

## What the receiver has to handle

- **Authentication to your endpoint is HTTP basic**, from the `webhookUsername` / `webhookPassword`
  on the rule. That is the only mechanism.
- **There is no signature.** No HMAC header, no shared-secret signing. Treat the basic credential as
  the whole of your authenticity check and terminate it on a private, TLS-only path.
- **Retry, ordering and replay behaviour are undocumented.** Assume at-most-once and reconcile
  against `GET /alarms` on a schedule.
- **The delivered payload has no schema in the contract.** Log the first deliveries and build your
  parser from what actually arrives.

## The one streaming endpoint

`listenAuthEvents` — `GET /admin/events/{sseSessionId}` — server-sent events for authentication
events. It is the only `text/event-stream` response in all 4,138 published operations, and it covers
auth events only, not alarms.
