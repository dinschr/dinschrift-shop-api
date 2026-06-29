# Dinschrift Shop API – Machine-to-Machine B2B Fulfillment

The Dinschrift Shop API is a streamlined REST-style HTTP API for sending print-on-demand textile orders **from one machine to another**. 

This repository describes our **Shop-Centric** fulfillment model. Rather than dealing with complex image uploads, DPI calculations, and print coordinates, integrators simply order pre-approved designs directly from their Dinschrift B2B Merch Shops.

The focus is on:

- Keeping the integration **frictionless** for developers and low-code tools (Zapier, Make, n8n, etc.).
- Utilizing a **Shop-Centric workflow** (Order by `design_hash` instead of raw image data).
- Matching **Dinschrift’s internal workflow** (SKU-based, automated production).

> **Status** > This V2 specification is currently in **beta**.  
> Do not rely on it in production without a direct agreement with Dinschrift.

---

## Who is this for?

This repository is aimed at:

- Integrators who want to send **direct B2B POD orders** to Dinschrift.
- Developers building:
  - Custom internal merch tools,
  - Staff uniform automations,
  - Low-code workflows (e.g., a webhook from Shopify to Zapier to Dinschrift).

It is designed to abstract away the complexity of print-on-demand, allowing you to focus purely on order fulfillment.

---

## Documentation

To keep things simple, all necessary documentation is consolidated right here. No subfolders to click through.

### 🔐 Authentication
All requests must include your API credentials in the headers. You can generate these in your [Dinschrift Dashboard](https://beta.dinschrift.ch/account/api-credentials).

```http
X-Shop-ID: 8a7b6c5d
X-Private-Key: sk_live_...
```

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
    "preview_url": "https://storage.googleapis.com/din_order_previews/a1b2c3d4e5f6g7h8_front.png"
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

*(Detailed response schema coming soon).*

---

## Scope

This repo contains:

- The data model for **Shop-Centric automated orders**.
- A conceptual design for a **machine-to-machine order API**:
  - submit orders,
  - fetch shop designs,
  - read order status.

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
```
