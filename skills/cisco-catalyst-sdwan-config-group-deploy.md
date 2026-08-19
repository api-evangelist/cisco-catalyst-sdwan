---
name: cisco-catalyst-sdwan-config-group-deploy
description: Provision devices with the UX 2.0 model — create a configuration group and its feature profiles, associate devices, resolve device variables, and deploy.
api: Cisco Catalyst SD-WAN Manager API
generated: '2026-08-19'
method: generated
source: openapi/cisco-catalyst-sdwan-ux-2-0-configuration-openapi.json, openapi/cisco-catalyst-sdwan-feature-profiles-sd-wan-system-openapi.json
operations:
  - CreateConfigGroup
  - GetConfigGroupBySolution
  - GetConfigGroup
  - EditConfigGroup
  - CreateSdwanSystemFeatureProfile
  - CreateSdwanTransportFeatureProfile
  - CreateSdwanServiceFeatureProfile
  - CreateConfigGroupAssociation
  - GetConfigGroupAssociation
  - fetchConfigGroupDeviceVariables
  - createConfigGroupDeviceVariables
  - deployConfigGroup
  - findRunningTasks
  - findStatus
  - CreateAaaProfileParcelForSystem
  - CreateLanVpnProfileParcelForService
  - GetSdwanServiceDhcpServerParcelSchemaBySchemaType
---

# Deploy a UX 2.0 configuration group

UX 2.0 replaces device templates with **configuration groups** made of **feature profiles**, which
are made of **parcels**. This is the provisioning path on current releases.

> **These are writes.** There is no idempotency key anywhere in this API and no dry-run. A retried
> POST creates a second object. Capture the returned id on every step and check before you retry.

## 0. Authenticate

`Authorization: Bearer <jwt>` plus `X-XSRF-TOKEN`. Writes here need
`Feature Profile > … -write` and `Template Deploy-write` roles; consult
`scopes/cisco-catalyst-sdwan-scopes.yml`.

## 1. Build the feature profiles

`CreateSdwanSystemFeatureProfile` — `POST /v1/feature-profile/sdwan/system`

`CreateSdwanTransportFeatureProfile` — `POST /v1/feature-profile/sdwan/transport`

`CreateSdwanServiceFeatureProfile` — `POST /v1/feature-profile/sdwan/service`

Each returns a profile id. Then add parcels beneath it — for example
`CreateAaaProfileParcelForSystem` (`POST /v1/feature-profile/sdwan/system/{systemId}/aaa`) or
`CreateLanVpnProfileParcelForService` (`POST /v1/feature-profile/sdwan/service/{serviceId}/lan/vpn`).

Every parcel type publishes its own JSON Schema at a `/schema` endpoint — for example
`GetSdwanServiceDhcpServerParcelSchemaBySchemaType`
(`GET /v1/feature-profile/sdwan/service/dhcp-server/schema?schemaType=post`). **Fetch the schema
first and build the payload from it.** The parcel payloads are large and heavily enumerated; guessing
them does not work.

## 2. Create the configuration group

`CreateConfigGroup` — `POST /v1/config-group`

Reference the profile ids from step 1. `GetConfigGroupBySolution` (`GET /v1/config-group`) lists what
already exists; `GetConfigGroup` (`GET /v1/config-group/{configGroupId}`) reads one back.

## 3. Associate devices

`CreateConfigGroupAssociation` — `POST /v1/config-group/{configGroupId}/device/associate`

`GetConfigGroupAssociation` — `GET /v1/config-group/{configGroupId}/device/associate`

## 4. Resolve device variables

`fetchConfigGroupDeviceVariables` — `POST /v1/config-group/{configGroupId}/device/variables`

`createConfigGroupDeviceVariables` — `PUT /v1/config-group/{configGroupId}/device/variables`

`getConfigGroupDeviceVariables` — `GET /v1/config-group/{configGroupId}/device/variables`

The fetch returns the variables the associated devices still need. Supply every one of them before
deploying, or the deploy fails per-device.

## 5. Deploy

`deployConfigGroup` — `POST /v1/config-group/{configGroupId}/device/deploy`

This pushes configuration to live network devices. It is asynchronous — the response is a task id.
Poll the task with `findRunningTasks` (`GET /device/action/status/tasks`) or `findStatus` (`GET /device/action/status/{processId}`) before declaring success.

## 6. Verify

Re-run the fabric inventory skill. Confirm control connections and BFD came back up on every device
you touched.

## Notes

- The `Feature Profiles - SD-WAN Transport`, `SD-WAN Service` and `SD-Routing` specifications are the
  three largest in this profile (8.2 MB, 7.8 MB and 17.5 MB) because every parcel schema is inlined.
  Load only the one you need.
- SD-Routing uses the same shape at `/v1/feature-profile/sd-routing/...` for routers managed without
  the SD-WAN overlay.
