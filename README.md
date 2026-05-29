# n8n Google Review Automation

An n8n workflow that automatically requests Google reviews from customers after a completed sale or service. Built for local businesses that want a reliable way to increase review volume without manually following up with every customer.

Works well for car dealerships, detailers, mechanics, cleaning businesses, home services, and small retail shops.

---

## What This Workflow Does

After a sale is marked as completed, the workflow:

1. Validates the incoming payload and checks that the customer has not opted out
2. Waits 2 days, then sends an initial review request SMS
3. Checks whether the SMS was delivered successfully
4. Waits 5 more days, re-checks opt-out status, then sends a reminder SMS
5. Notifies the business owner by email on success or failure

The goal is to automate follow-up and increase review volume with minimal manual work.

---

## Workflow Overview

```text
Sale Completed Trigger
        |
Check: Valid sale_id + Not Opted Out
        |
    [Pass]
        |
Wait 2 Days
        |
Send Review Request SMS
        |
SMS Delivered?
   |          |
[Yes]        [No]
   |          |
Wait 5 Days  Notify Owner of Failure
   |
Still Not Opted Out?
   |          |
[Yes]        [No - stop]
   |
Send Reminder SMS
   |
SMS Delivered?
   |          |
[Yes]        [No]
   |          |
Notify Owner  Notify Owner of Failure
```

---

## Requirements

- n8n (self-hosted or cloud)
- Twilio account for SMS sending
- A Google review link for your business
- Gmail account for owner notifications (optional)

---

## Setup

### 1. Import the workflow

Import `workflow.json` into n8n via Settings > Import workflow.

### 2. Connect Twilio

Open each Twilio node and add your credentials:

- Account SID
- Auth Token
- Twilio phone number (the `from` field)

### 3. Set a fallback Google review link

The workflow uses the `review_link` field from the incoming payload if provided. If not, it falls back to a hardcoded URL. Replace the placeholder in both SMS nodes:

```text
https://g.page/r/YOUR-REVIEW-LINK/review
```

Passing `review_link` dynamically in the webhook payload is recommended for multi-location setups.

### 4. Connect Gmail

Open the Gmail nodes and connect your business Gmail account via OAuth2. Update the `toEmail` field with the owner's address.

### 5. Activate the workflow

Once credentials are connected, activate the workflow and send a test payload to the webhook endpoint.

---

## Webhook Payload

Send a POST request to `/webhook/sale-completed` with the following fields:

```json
{
  "sale_id": "sale_abc123",
  "customer_name": "John Smith",
  "phone": "+15551234567",
  "service": "Interior Detail",
  "review_link": "https://g.page/r/YOUR-REVIEW-LINK/review",
  "opted_out": false
}
```

| Field | Required | Description |
|---|---|---|
| `sale_id` | Yes | Unique identifier for deduplication. Workflow will not proceed if missing. |
| `customer_name` | Yes | Used to personalize SMS messages. |
| `phone` | Yes | Customer's mobile number in E.164 format. |
| `service` | Yes | Service or product purchased, used in the SMS body. |
| `review_link` | No | Per-location Google review URL. Falls back to hardcoded value if omitted. |
| `opted_out` | No | Set to `true` to suppress all messages for this customer. Defaults to `false`. |

---

## Example SMS Messages

Initial request (sent after 2 days):

```text
Hey John - Thanks for choosing us for your Interior Detail.
We'd really appreciate a quick Google review:
https://g.page/r/YOUR-REVIEW-LINK/review
```

Reminder (sent after 5 more days):

```text
Hey John - Just a quick follow-up. If you enjoyed your experience
with us, a Google review would mean a lot.
https://g.page/r/YOUR-REVIEW-LINK/review
Reply STOP to opt out.
```

---

## Error Handling

- If either SMS fails to send, the sequence stops and the owner receives an email with the customer name, phone number, sale ID, and error details for manual follow-up.
- Owner success notification is only sent after the reminder SMS is confirmed delivered, not unconditionally at the end of the flow.

---

## Opt-Out Handling

The workflow checks the `opted_out` field on entry and again before the reminder SMS. To comply with SMS regulations (TCPA and similar), configure a Twilio inbound message handler that sets `opted_out: true` in your system when a customer replies STOP. Your CRM or webhook source should reflect this before the reminder fires.

---

## Deduplication

The workflow requires a non-empty `sale_id` field in the payload. If your source system retries webhook delivery, the same `sale_id` will pass through the check node again. For stricter deduplication, add a database lookup (e.g., via a Postgres or Redis node) to record processed sale IDs and reject duplicates before the wait step.

---

## Adjusting Timing

Both wait nodes can be changed to any duration. Open the node and update the `amount` and `unit` fields. Common adjustments:

- For same-day services (e.g., car wash): reduce first wait to a few hours
- For longer projects (e.g., renovation): increase first wait to 4-7 days

---

## Notes

- The workflow uses `continueOnFail: true` on Twilio nodes so SMS errors are caught by the IF check rather than crashing the execution.
- The review link can be passed dynamically per sale, making the workflow suitable for businesses with multiple locations.
- Works with any CRM, POS system, or booking platform that can POST to a webhook endpoint.

---

## Ideas for Future Improvements

- WhatsApp support via Twilio's WhatsApp API
- AI-personalized message copy based on service type
- Google Business API integration to detect if a review was already left and skip the reminder
- Slack or SMS notifications to the owner instead of email
- CRM write-back to log sequence completion
- Stricter deduplication using a database node

---

## Repo Structure

```text
.
├── workflow.json
├── README.md
├── screenshots/
└── .env.example
```

---

## License

MIT