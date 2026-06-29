# Dinschrift Shop API – Store Fulfillment

The Dinschrift Shop API is a REST HTTP API for automating print-on-demand textile orders. 

This API is designed exclusively for ordering from your pre-configured custom stores within the Dinschrift Dashboard. By utilizing a **Shop-Centric** fulfillment model, you simply order pre-approved designs using a `design_hash` rather than dealing with raw image uploads, DPI calculations, or print coordinates.

> **Status** 
> This V2 specification is currently in **beta**.  
> Do not rely on it in production without a direct agreement with Dinschrift.

---

## Who is this for?

This repository is aimed at developers and integrators who want to route orders to Dinschrift via:
- Custom internal merch tools
- Staff uniform automations
- Workflow automations (e.g., a webhook from Shopify to Zapier to Dinschrift)

---

## Documentation

All API documentation and endpoints are consolidated below.

### 🔐 Authentication
All requests must include your API credentials in the headers. You can generate these in your [Dinschrift Dashboard](https://beta.dinschrift.ch/account/api-credentials).

```http
X-Shop-ID: 8a7b6c5d
X-Private-Key: sk_live_...
```

### 💳 Billing & Payments
The Dinschrift Shop API operates on a fully automated billing cycle. To successfully submit an order, you must have a valid credit card saved in your [Dinschrift Dashboard](https://beta.dinschrift.ch/account/payment-methods).

* When you submit an order via the `POST /api/v2/orders` endpoint, your default saved card is automatically charged.
* Credit card data is never passed through the API; it relies entirely on the secure token already saved in your account via our Swiss partner Saferpay.

### ⚠️ Errors
When an API request fails, you will receive standard HTTP status codes along with a JSON response detailing the issue.

Common error codes:
* `400 Bad Request`: Missing required fields (e.g., missing `design_hash` or invalid SKU).
* `401 Unauthorized`: Invalid or missing `X-Private-Key` or `X-Shop-ID`.
* `402 Payment Required`: The order could not be processed. This happens if no default credit card is saved in your dashboard, the card has expired, or the payment was declined by the bank.
* `404 Not Found`: The requested `shop_id` or `design_hash` does not exist or does not belong to your account.

**Example Error Response:**
```json
{
  "error": "payment_failed",
  "message": "Payment declined. Please update your saved credit card in the Dinschrift Dashboard.",
  "status_code": 402
}
```

---

### 1️⃣ Fetch Available Designs
Retrieve all approved designs currently assigned to your specific Client Shop. This endpoint returns the `design_hash` required to submit an order.

**`GET /api/v2/shops/{shop_id}/designs`**

**Example Response:**
```json
[
  {
    "design_hash": "a1b2c3d4e5f6g7h8",
    "design_name": "Summer Vibes Drop - Front Logo",
    "base_cost": 24.50,
    "retail_price": 39.50,
    "preview_url": "[https://storage.googleapis.com/din_order_previews/a1b2c3d4e5f6g7h8_front.png](https://storage.googleapis.com/din_order_previews/a1b2c3d4e5f6g7h8_front.png)"
  }
]
```

### 2️⃣ Submit an Order
Submit a fulfillment order using the known `design_hash` and standard SKUs. 

**`POST /api/v2/orders`**

**Example Request:**
```json
{
  "external_order_reference": "SHOP-12345",
  "shop_id": "8a7b6c5d",
  "is_express": false,
  "items": [
    {
      "design_hash": "a1b2c3d4e5f6g7h8",
      "sizes": [
        { "sku": "STSU168C651S", "quantity": 1 },
        { "sku": "STSU168C651M", "quantity": 2 }
      ]
    }
  ],
  "shipping_address": {
    "name": "Jane Doe",
    "street": "Example Street 1",
    "city": "Zürich",
    "zip": "8000",
    "country": "CH"
  }
}
```

### 3️⃣ Check Order Status
Poll for tracking numbers, expected delivery dates, and production status.

**`GET /api/v2/orders/{id}`**

**Example Response:**
```json
{
  "order_id": "101747308500210612",
  "external_order_reference": "SHOP-12345",
  "status": "production",
  "expected_delivery_date": "2026-07-05",
  "tracking": {
    "carrier": "Swiss Post",
    "tracking_number": "99.12.345678.12345678",
    "tracking_url": "[https://service.post.ch/ekp-web/ui/list?ids=99.12.345678.12345678](https://service.post.ch/ekp-web/ui/list?ids=99.12.345678.12345678)"
  },
  "totals": {
    "subtotal": 82.00,
    "express_surcharge": 0.00,
    "grand_total": 88.64,
    "vat_rate": 8.1
  }
}
```

---

## Scope

This repo contains:
- The data model for **Shop-Centric automated orders**.
- A conceptual design for the order API.

This repo does **not** contain:
- Backend source code or deployment scripts,
- API keys, secrets or live credentials,
- Billing or payment processing implementation.

Commercial use with Dinschrift as the print provider always requires a separate commercial agreement.

---

## Stability and changes

Because this spec is in **beta**, we may:
- add new fields (in a backwards-compatible way),
- clarify behaviour and error codes,
- add new endpoints (e.g. webhooks).

Breaking changes, if ever needed, will be introduced via **versioning** and documented in this repo.

---

## License

Unless otherwise stated in individual files, the written specification in this repository is made available under the **MIT License**.

You are free to:
- read, implement and experiment with this spec,
- use it internally or with Dinschrift under a commercial agreement.
