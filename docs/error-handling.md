# Error Handling

This guide covers HTTP status codes, error response formats, and retry strategies for the AlayaCare and Marketplace APIs used in this integration.

## HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| `200` | Success | Request completed. Parse the response body. |
| `201` | Created | Resource created (visits, messages). Parse the response for the new resource ID. |
| `400` | Bad Request | Invalid request body or parameters. Check field names, types, and required fields. Do not retry without fixing the request. |
| `401` | Unauthorized | Invalid or missing API credentials. Verify your API key and secret. |
| `403` | Forbidden | Valid credentials but insufficient permissions. Confirm your integration has the External API role. |
| `404` | Not Found | The resource does not exist (e.g., invalid offer ID, referral ID, or client ID). Verify the ID is correct. |
| `409` | Conflict | The operation conflicts with the current resource state (e.g., accepting an already-accepted offer, processing an already-processed referral). Check the resource status before retrying. |
| `422` | Unprocessable Entity | The request is well-formed but semantically invalid (e.g., missing required referral fields, invalid date range). Review the error message for details. |
| `429` | Too Many Requests | Rate limit exceeded. Back off and retry after the interval indicated in the `Retry-After` header, or use exponential backoff starting at 1 second. |
| `500` | Internal Server Error | Server-side failure. Retry with exponential backoff. If persistent, contact your AlayaCare integration contact. |
| `502/503` | Bad Gateway / Service Unavailable | Temporary infrastructure issue. Retry with exponential backoff. |

## Error Response Format

API errors return a JSON body with an error description:

```json
{
  "error": "bad_request",
  "message": "Field 'alayacare_client_id' is required when override_client is specified."
}
```

Some endpoints may return a simpler format:

```json
{
  "message": "Offer has already been accepted."
}
```

Always check the HTTP status code first, then parse the response body for details.

## Retry Strategy

### Idempotency

- `GET` requests are safe to retry at any time.
- `POST` actions on offers (`accept`, `refuse`) and referrals (`process`) are **not idempotent** — retrying after a `200` may produce a `409`. Always check the resource status before retrying.
- `POST` for messages is **not idempotent** — a retry will send a duplicate message. Track sent message IDs to avoid duplicates.
- `PUT` for visits is idempotent if the same payload is sent.

### Exponential Backoff

For `429`, `500`, `502`, and `503` responses, use exponential backoff:

1. Wait 1 second, then retry.
2. If the retry fails, wait 2 seconds.
3. Double the wait each time, up to a maximum of 60 seconds.
4. After 5 consecutive failures, stop retrying and alert your operations team.

### SQS Event Redelivery

If your event consumer fails to process an SQS message, the message returns to the queue after the visibility timeout. Design your consumer to handle duplicate deliveries:

- Use the event's unique identifier to deduplicate.
- Make your processing logic idempotent where possible (e.g., check if a referral is already processed before calling the process endpoint).

## Common Failure Scenarios

| Scenario | Likely Status | Resolution |
|----------|--------------|------------|
| Accepting an offer that was already accepted | `409` | Fetch the offer status first; skip if already accepted. |
| Accepting an offer that was closed, expired, or fulfilled | `409` or `400` | Fetch the offer to check its current status. Handle non-actionable states gracefully. |
| Processing a referral that was already processed | `409` | Fetch the referral status; skip if already processed. |
| Processing a referral that was cancelled by the demand agency | `400` or `409` | Handle `ReferralDemandCancelled` events to remove cancelled referrals from your queue. |
| Client search returns no results | `200` with empty list | Proceed with Strategy A (new client + new service). |
| Uploading a file with mismatched `client_id` in the path | `400` or `404` | Ensure the `client_id` in the URL matches the client associated with the referral. |
| Sending a message with an invalid `alayamarket_sequence_id` | `404` | Verify the sequence ID was obtained from a valid referral. |
| Visit creation with invalid date range | `422` | Ensure `start_at` is before `end_at` and both are valid UTC datetimes. |

## Rate Limits

AlayaCare APIs enforce rate limits to protect platform stability. When you exceed the limit, the API returns `429 Too Many Requests`.

- Check the `Retry-After` response header for the number of seconds to wait before retrying.
- If no `Retry-After` header is present, use exponential backoff starting at 1 second.
- Design your integration to stay well under rate limits by batching reads and avoiding tight polling loops.
- For SQS-triggered workflows, process events sequentially per referral/offer rather than parallelizing all API calls at once.

> Specific rate limit thresholds are environment-dependent and may change. Contact your AlayaCare integration contact for current limits if you're running into `429` responses.

---

**Back to:** [README](../README.md) | [Environment Setup](setup.md)
