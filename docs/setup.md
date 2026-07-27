# Environment Setup

This guide covers provisioning, authentication, and event configuration for 3rd-party partner integration.

## Environments

| Environment | Base URL | Notes |
|-------------|----------|-------|
| UAT | `https://{partner}.uat.alayacare.com` | Integration development and testing |
| Production | `https://{partner}.alayacare.com` | Live environment |

Replace `{partner}` with your provisioned subdomain.

Throughout this guide, all URLs use the placeholder `$ACCLOUD_URL` to represent your environment's base URL.

## Supply Organization

Each partner is provisioned as a **Supply Organization** on Marketplace with a dedicated AlayaCare branch.

| Field | Example Value |
|-------|---------------|
| Marketplace Supply Org | `{PARTNER_NAME}` |
| Branch ID | `{BRANCH_ID}` |
| Branch URL | `$ACCLOUD_URL/` |

Your AlayaCare contact will provide these values during onboarding.

## Authentication

### API Integration Credentials

An **External API** integration is created on your branch for programmatic access.

- **Location:** `$ACCLOUD_URL/#/system-settings/external-integrations`
- **Role:** External API
- **Authentication:** HTTP Basic Auth using the integration's API key and secret

```
Authorization: Basic base64({api_key}:{api_secret})
```

All API calls in this guide assume Basic Auth unless otherwise noted.

### Employee User

An employee user with the **External API** role is associated with your integration for audit purposes.

- **Location:** `$ACCLOUD_URL/#/employees/{employee_id}/overview`
- **Role:** External API

## Event Subscriptions

Your integration receives real-time notifications from Marketplace via AlayaCare's external event system (SQS queues).

- **Configuration:** `$ACCLOUD_URL/#/system-settings/external-events/sqs-queues`
- **Filter by group:** `marketplace`

Subscribe using the **ACC event type** strings below. These are the values shown in the SQS queue UI. They are not the same as AsyncAPI operation IDs (`sub_inbox_*`).

### Required Event Subscriptions (supply partners)

| ACC event type (subscribe) | Status | Description | Example subtype | AsyncAPI channel (schema) |
|----------------------------|--------|-------------|-----------------|---------------------------|
| `marketplace-demand-referral` | Live | Referrals sent by demand agencies | `marketplace-demand-referral-created` | [inbox-referrals](https://alayacare.github.io/alayamarket-external-docs/docs/offers/asyncapi.external.offers/#operation-send-sub_inbox_referrals) |
| `marketplace-demand-message` | Live | Messages sent by demand agencies | `marketplace-demand-message-sent` | [inbox-messages](https://alayacare.github.io/alayamarket-external-docs/docs/offers/asyncapi.external.offers/#operation-send-sub_inbox_messages) |
| `marketplace-demand-offer` | Not live yet | Offers from demand agencies | — | [inbox-offers](https://alayacare.github.io/alayamarket-external-docs/docs/offers/asyncapi.external.offers/#operation-send-sub_inbox_offers) |

When an event fires, your SQS queue receives a payload with a `subtype` and identifiers you use to fetch full details via the corresponding API (see workflow guides). Use the AsyncAPI links for payload schema and internal `event_name` values (`ReferralCreated`, `MessageDemandSent`, and so on) — not as the subscription name in ACC.

> **Offers:** ACC registration for offer events is not available in the SQS UI yet. Keep following the [Offers](workflows/offers.md) API workflow; subscribe to `marketplace-demand-offer` when your AlayaCare contact confirms it is live.

### Event naming

Marketplace ACC events use this pattern:

1. **Subscription `event_type`:** `marketplace-{actor}-{resource}` (singular resource noun).
2. **Payload `subtype`:** `marketplace-{actor}-{resource}-{verb}`.
3. **Actor** = who initiated the action. As a **supply** partner you subscribe to **`marketplace-demand-*`** for inbox traffic (demand acted; you receive it).
4. Do not confuse Marketplace events with the separate **Intake** group (`intake-referral-created`, `intake-offer-created`, legacy `referral-processed`). Those are Intake auto-processing events, not Marketplace partner inbox subscriptions.

| Who acted | ACC prefix | Who typically subscribes |
|-----------|------------|--------------------------|
| Demand | `marketplace-demand-*` | Supply (your org) |
| Supply | `marketplace-supply-*` | Demand agencies |

Example live payload fields for a demand-sent message:

```json
{
  "subtype": "marketplace-demand-message-sent",
  "alayamarket_message_id": "…",
  "alayamarket_sequence_id": "…",
  "category": "comment",
  "branch_id": 1000
}
```

## Integration Checklist

Before starting development, confirm:

- [ ] Provisioned branch URL and credentials received
- [ ] API integration created and tested (try `GET $ACCLOUD_URL/ext/api/v2/patients/clients?page=1&count=1`)
- [ ] SQS queue subscribed to `marketplace-demand-referral` and `marketplace-demand-message` (group: `marketplace`)
- [ ] Event delivery confirmed in your queue

## Next Steps

Once your environment is set up, work through the integration workflows:

1. [Offers](workflows/offers.md) — Receive and respond to offers
2. [Referrals](workflows/referrals.md) — Receive and process referrals
3. [Messages](workflows/messages.md) — Exchange messages and attachments
4. [Visits](workflows/visits.md) — Report visit scheduling and status
