# n8n Google Review Automation

Simple n8n workflow for automatically requesting Google reviews from customers after a completed sale or service.

Built originally for local businesses that wanted an easy way to get more reviews without manually texting every customer.

Works well for:

* car dealerships
* detailers
* mechanics
* cleaning businesses
* home services
* small retail shops

---

## What This Workflow Does

After a customer is marked as completed:

1. waits 2 days
2. sends a review request SMS
3. waits a few more days
4. sends a reminder message
5. notifies the business owner

The goal is just to automate follow-up and increase review volume with almost no manual work.

---

## Workflow Overview

```text
Sale Completed
    ↓
Wait 2 Days
    ↓
Send Review Request SMS
    ↓
Wait 5 Days
    ↓
Send Reminder SMS
    ↓
Notify Owner
```

---

## Requirements

* n8n
* Twilio account (for SMS)
* Google review link
* Gmail account (optional for notifications)

---

## Setup

### 1. Import the workflow

Import `workflow.json` into n8n.

### 2. Connect Twilio

Add your:

* Account SID
* Auth Token
* Twilio phone number

inside the Twilio nodes.

### 3. Replace the Google review link

Search for:

```text
https://g.page/r/YOUR-REVIEW-LINK/review
```

and replace it with your own review URL.

### 4. Activate workflow

Once credentials are connected, activate the workflow and send test data to the webhook.

---

## Example Payload

```json
{
  "customer_name": "John Smith",
  "phone": "+15551234567",
  "service": "Interior Detail"
}
```

---

## Example SMS

```text
Hey John 👋

Thanks for choosing us for your Interior Detail.

We'd really appreciate a quick Google review:
https://g.page/r/YOUR-REVIEW-LINK/review
```

---

## Notes

* The timing delays can be adjusted easily.
* You can swap SMS for WhatsApp if preferred.
* Works with pretty much any CRM or booking system that can send webhook data.
* I kept the workflow intentionally simple so it's easy to modify.

---

## Ideas for Improvements

Some things I may add later:

* WhatsApp support
* AI-personalized messages
* Google review detection
* Slack notifications
* CRM integrations
* QR code generator

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
