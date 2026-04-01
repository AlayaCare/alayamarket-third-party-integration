# Messages

Messages allow your organization to communicate with demand agencies through Marketplace. You can send and receive text messages and file attachments within the context of a referral sequence.

## Prerequisites

- [Environment setup](../setup.md) complete
- `sub_inbox_messages` event subscription active

---

## Step 1: Receive Message Events

When a demand agency sends a message, your SQS queue receives a `sub_inbox_messages` event.

- **AsyncAPI reference:** [sub_inbox_messages](https://alayacare.github.io/alayamarket-external-docs/docs/offers/asyncapi.external.offers/#operation-send-sub_inbox_messages)

---

## Step 2: Fetch Messages

Retrieve messages for a given referral sequence.

* **URL:** `GET $ACCLOUD_URL/api/v1/alayamarket/messages/supply`

* **Query parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `branch_id` | Yes | Your branch ID (constant) |
| `alayamarket_sequence_id` | Yes | The sequence ID from the referral |
| `items_per_page` | No | Number of results per page (default: 10) |
| `page` | No | Page number (default: 1) |
| `sort_order` | No | `asc` or `desc` (default: `desc`) |

* **Example:**

```
GET $ACCLOUD_URL/api/v1/alayamarket/messages/supply?branch_id={BRANCH_ID}&alayamarket_sequence_id={sequence_id}&items_per_page=10&page=1&sort_order=desc
```

---

## Step 3: Send a Message

Send a text message to the demand agency within a referral sequence.

* **URL:** `POST $ACCLOUD_URL/api/v1/alayamarket/messages/supply`

* **Content-Type:** `multipart/form-data`

* **Form fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `branch_id` | Yes | Your branch ID (constant) |
| `alayamarket_sequence_id` | Yes | The sequence ID for the referral |
| `category` | Yes | `comment` |
| `sender_type` | Yes | `supply` |
| `text` | Yes | Message body (up to 5,000 characters) |
| `author` | Yes | Display name of the sender |

* **Example (curl):**

```bash
curl -X POST "$ACCLOUD_URL/api/v1/alayamarket/messages/supply" \
  -u "$API_KEY:$API_SECRET" \
  -F "branch_id={BRANCH_ID}" \
  -F "alayamarket_sequence_id={sequence_id}" \
  -F "category=comment" \
  -F "sender_type=supply" \
  -F "text=We have reviewed the referral and will begin care on Monday." \
  -F "author=Jane Doe"
```

See [examples/payloads/send_message.json](../../examples/payloads/send_message.json) for field reference.

---

## Step 4: Upload a Client Attachment

Upload a file to the client's attachments in AlayaCare. The file is stored under the client's Marketplace Documents folder.

* **API reference:** [files-api-external](https://app.swaggerhub.com/apis/AlayaCare/files-api-external/1.0.0#/File/post_client__id___path_)

* **URL:** `POST $ACCLOUD_URL/ext/api/v2/files/client/{client_id}/{file_path}`

* **Content-Type:** `multipart/form-data`

* **Form fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `file` | Yes | The file binary |

* **Example (curl):**

```bash
curl -X POST "$ACCLOUD_URL/ext/api/v2/files/client/{client_id}/marketplace-documents/care-plan.pdf" \
  -u "$API_KEY:$API_SECRET" \
  -F "file=@/path/to/care-plan.pdf"
```

* **Notes:**
  * The `client_id` in the URL path and the client ID embedded in the `file_path` must match — otherwise the file may appear under the wrong client
  * The file path is associated with a service ID. To find the correct path, look up the sequence by service ID (see below)

### Find the File Path via Sequence

* **URL:** `GET $ACCLOUD_URL/api/v1/alayamarket/inbox/sequences/by_service_id/{service_id}`

* **Notes:**
  * Returns sequence metadata including the file path structure for the associated client and service

---

## Step 5: Send a Message with Attachment

Send a message that includes a file attachment within the referral sequence.

* **URL:** `POST $ACCLOUD_URL/api/v1/alayamarket/messages/supply`

* **Content-Type:** `multipart/form-data`

* **Form fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `branch_id` | Yes | Your branch ID (constant) |
| `alayamarket_sequence_id` | Yes | The sequence ID for the referral |
| `category` | Yes | `client_attachment` |
| `sender_type` | Yes | `supply` |
| `text` | Yes | Message body (up to 5,000 characters) |
| `author` | Yes | Display name of the sender |
| `file` | Yes | The file binary |

* **Example (curl):**

```bash
curl -X POST "$ACCLOUD_URL/api/v1/alayamarket/messages/supply" \
  -u "$API_KEY:$API_SECRET" \
  -F "branch_id={BRANCH_ID}" \
  -F "alayamarket_sequence_id={sequence_id}" \
  -F "category=client_attachment" \
  -F "sender_type=supply" \
  -F "text=Attached is the signed care plan." \
  -F "author=Jane Doe" \
  -F "file=@/path/to/signed-care-plan.pdf"
```

See [examples/payloads/send_message_attachment.json](../../examples/payloads/send_message_attachment.json) for field reference.

---

## Step 6: Retrieve an Attachment

### Option A: Retrieve a Client Attachment by Path

* **URL:** `GET $ACCLOUD_URL/ext/api/v2/files/client/{client_id}/{file_path}`

* **Notes:**
  * Use the same path structure from Step 4
  * The file path can be discovered from the sequence endpoint (Step 4 note)

### Option B: Retrieve a Message Attachment by File ID

Use the file ID from a message (fetched in Step 2) to get a preview/download link.

* **URL:** `GET $ACCLOUD_URL/api/v1/alayamarket/messages/supply/files/{file_id}/preview?branch_id={BRANCH_ID}`

---

## Sequence

```mermaid
sequenceDiagram
    participant DA as Demand Agency
    participant MP as Marketplace
    participant AC as ACCloud Bridge
    participant 3P as Your System

    DA->>MP: Send message
    MP->>AC: Deliver message
    AC-->>3P: sub_inbox_messages event (SQS)
    3P->>AC: GET messages for sequence
    AC-->>3P: Message list

    3P->>AC: POST send message
    AC->>MP: Relay message
    MP->>DA: Deliver message

    3P->>AC: POST upload attachment
    3P->>AC: POST send message with attachment
    AC->>MP: Relay attachment
    MP->>DA: Deliver attachment
```

---

**Next:** [Visits](visits.md) — Report visit scheduling and status to demand agencies
