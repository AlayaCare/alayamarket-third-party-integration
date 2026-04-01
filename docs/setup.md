# Environment Setup

This guide covers provisioning, authentication, and event configuration for 3rd-party partner integration.

## Environments

| Environment | Base URL | Notes |
|-------------|----------|-------|
| UAT | `https://{partner}.uat.alayacare.com` | Integration development and testing |
| Production | `https://{partner}.alayacare.com` | Live environment |

Replace `{partner}` with your provisioned subdomain (e.g. `myreferrals`).

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

## Event Subscriptions (Work in Progress)

Your integration receives real-time notifications from Marketplace via AlayaCare's external event system (SQS queues).

- **Configuration:** `$ACCLOUD_URL/#/system-settings/external-events/sqs-queues`

### Required Event Subscriptions

| Event | Description | AsyncAPI Reference |
|-------|-------------|-------------------|
| `sub_inbox_offers` | New offers from demand agencies | [AsyncAPI spec](https://alayacare.github.io/alayamarket-external-docs/docs/offers/asyncapi.external.offers/#operation-send-sub_inbox_offers) |
| `sub_inbox_referrals` | New referrals from demand agencies | [AsyncAPI spec](https://alayacare.github.io/alayamarket-external-docs/docs/offers/asyncapi.external.offers/#operation-send-sub_inbox_referrals) |
| `sub_inbox_messages` | New messages from demand agencies | [AsyncAPI spec](https://alayacare.github.io/alayamarket-external-docs/docs/offers/asyncapi.external.offers/#operation-send-sub_inbox_messages) |

When an event fires, your SQS queue receives a payload containing identifiers you can use to fetch the full details via the corresponding API (see workflow guides).

## Integration Checklist

Before starting development, confirm:

- [ ] Provisioned branch URL and credentials received
- [ ] API integration created and tested (try `GET $ACCLOUD_URL/ext/api/v2/patients/clients?page=1&count=1`)
- [ ] SQS queues configured for all three event types
- [ ] Event delivery confirmed in your queue

## Next Steps

Once your environment is set up, work through the integration workflows:

1. [Offers](workflows/offers.md) — Receive and respond to offers
2. [Referrals](workflows/referrals.md) — Receive and process referrals
3. [Messages](workflows/messages.md) — Exchange messages and attachments
4. [Visits](workflows/visits.md) — Report visit scheduling and status
