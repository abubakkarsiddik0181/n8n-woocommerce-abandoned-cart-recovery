# E-commerce Abandoned Cart Recovery — n8n + WooCommerce

<img width="1155" height="570" alt="n8n-woocommerce-abandoned-cart-recovery" src="https://github.com/user-attachments/assets/8f70a4e4-7f71-4b89-99bc-fede5156847f" />


An automated abandoned cart recovery system built with n8n that detects when a customer leaves without purchasing, sends a personalized reminder email after 30 minutes, and follows up with a unique 10% discount coupon after 24 hours — all tracked in Google Sheets.

---

## What It Does

- Detects cart abandonment via WooCommerce webhook in real time
- Saves customer and cart data to Google Sheets automatically
- Checks if the customer has already purchased before sending any email
- Sends a personalized reminder email 30 minutes after abandonment
- After 24 hours, generates a unique discount coupon via WooCommerce API and emails it
- Coupon is tied to the customer's email — only they can use it
- Coupon expires in 48 hours and has a single-use limit
- Updates Google Sheets with status: Pending, Recovered, or Lost
- Fully automated — no manual work required after setup

---

## Tech Stack

- **n8n** — Workflow automation
- **WooCommerce** — Order and coupon management via REST API
- **Abandoned Cart Lite** — WordPress plugin for cart abandonment detection
- **Gmail** — Sending reminder and discount emails via OAuth2
- **Google Sheets** — Lead tracking and status management
- **JavaScript (Code Node)** — Data cleaning, time calculations, coupon logic

---

## Workflow Overview

### Workflow 1 — Cart Intake and 30-Minute Reminder

```
WooCommerce Webhook (cart abandoned)
  → Code Node (clean and format data)
  → Google Sheets (save customer + cart info)
  → Wait 30 minutes
  → WooCommerce API (check if order already placed)
  → IF no order → Gmail (send reminder email) → Sheets (Reminder Sent: Yes)
  → IF order placed → Sheets (Status: Recovered)
```

### Workflow 2 — 24-Hour Discount Follow-up

```
Schedule Trigger (every 5 minutes)
  → Google Sheets (fetch rows where Reminder Sent: Yes, Discount Sent: No)
  → Loop Over Items
  → Code Node (check if 24 hours have passed since abandonment)
  → IF 24 hours passed → WooCommerce API (check for order)
      → IF no order → Code Node (generate unique coupon data)
          → WooCommerce API (create coupon)
          → Gmail (send discount email)
          → Sheets (Discount Sent: Yes)
      → IF order placed → Sheets (Status: Recovered)
  → IF not 24 hours yet → skip, loop continues
```

### Workflow 3 — Final Status Update (Lost or Recovered)

```
Schedule Trigger (every 5 minutes)
  → Google Sheets (fetch rows where Discount Sent: Yes)
  → Loop Over Items
  → Code Node (check if 72 hours have passed since abandonment)
  → IF 72 hours passed → WooCommerce API (final order check)
      → IF no order → Sheets (Status: Lost)
      → IF order placed → Sheets (Status: Recovered)
  → IF not 72 hours yet → skip
```

---

## Google Sheet Structure

| Customer Name | Email | Product | Cart Value | Status | Cart Abandoned Time | Reminder Sent | Discount Sent | Checkout Link |
|---|---|---|---|---|---|---|---|---|
| John Smith | john@example.com | Nike Air Max | $120 | Pending | 06/16/2026 01:11 AM | No | No | https://... |

Status values: `Pending` → `Recovered` or `Lost`

---

## Setup Instructions

### 1. WordPress and WooCommerce Setup

- Install WordPress and WooCommerce
- Install the **Abandoned Cart Lite for WooCommerce** plugin (by Tyche Softwares)
- In Abandoned Cart Lite settings, set cart abandoned cut-off time to `30` minutes
- Enable "Start tracking from Cart Page"

### 2. WooCommerce Webhook

- Go to WooCommerce → Settings → Advanced → Webhooks → Add Webhook
- Name: `Abandoned Cart`
- Status: `Active`
- Topic: `Cart: Abandoned after cut-off time`
- Delivery URL: your n8n Production Webhook URL
- Save

### 3. WooCommerce REST API

- Go to WooCommerce → Settings → Advanced → REST API → Add Key
- Set Permission to `Read/Write`
- Copy the Consumer Key and Consumer Secret

### 4. Enable Real Cron (Important)

The Abandoned Cart plugin uses WordPress cron, which only fires when someone visits the site. To ensure timely detection, set up a real cron job on your hosting:

```
*/5 * * * * wget -q -O /dev/null https://yoursite.com/wp-cron.php
```

This can be set up in cPanel → Cron Jobs.

### 5. Import Workflows into n8n

- Open your n8n instance
- Go to Workflows → Import from File
- Import `abandoned_cart_workflow.json`

### 6. Connect Credentials

| Credential | Used In |
|---|---|
| Gmail OAuth2 | Reminder email and discount email nodes |
| Google Sheets OAuth2 | All Google Sheets nodes |
| WooCommerce Basic Auth | HTTP Request nodes (Consumer Key + Secret) |

### 7. Update Placeholders

In the imported workflow, replace these placeholders:

- `YOUR_GOOGLE_SHEET_ID` — found in your Google Sheet URL
- `YOUR_WORDPRESS_SITE` — your WooCommerce store URL

### 8. Activate

- Activate the workflow
- Test by adding a product to cart, entering your email at checkout, and leaving without purchasing

---

## Coupon Logic

Each customer receives a unique coupon code generated from their email address plus a timestamp, for example: `SAVE10_JOHNDOE_1718500000000`

- 10% discount
- Single use only
- Restricted to the customer's email address
- Expires in 48 hours

---

## Author

**Abu Bakkar Siddik**
