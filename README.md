# Dinschrift Shop API

The Dinschrift Shop API enables automated M2M (Machine-to-Machine) and guest shop orders for pre-configured t-shirts available in the client dashboard or Merch-as-a-Service store. It provides a straightforward way to submit orders and receive real-time status updates via webhooks.

---

## 🔐 Authentication

All API requests must be authenticated using your API credentials. You can generate and manage these in the **API & Credentials** section of your Dinschrift Dashboard.

Pass your credentials in the headers of your requests:

```http
X-Shop-ID: <your_shop_id>
X-Private-Key: <your_private_key>
```

---

## 🛒 Ordering

Orders are submitted via a POST request to our API. This handles orders coming directly from your custom `shops.dinschrift.ch` storefronts or your own M2M integrations.

**Endpoint:**
`POST /api/v2/orders`

*(Note: Use the Payload Builder in your API dashboard to simulate and structure your order requests.)*

---

## 🪝 Webhooks & Status Updates

Instead of polling the system to check if a t-shirt has been printed or shipped, you can configure a Webhook URL. The system will automatically send a POST request to your URL whenever an order's status changes.

### Configuration
You can save your destination Webhook URL directly in your Dinschrift Dashboard under the **API & Credentials** section.

### Payload Structure
When an order status changes (e.g., to status `90`), your server will receive a JSON payload containing the updated order details. 

**Example Payload:**
```json
{
  "success": true,
  "data": {
    "orderId": "API-TEST-LIVE-4",
    "orderNumber": 54564,
    "orderDate": "2026-09-10T12:22:53.000Z",
    "statusCode": "90",
    "totalAmount": 36.25,
    "items": []
  } 
}
```    "totalAmount": 36.25,
    "items": []
  } 
}
```

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
  "message": "No default payment method found for this account."
}
```  "message": "Payment declined. Please update your saved credit card in the Dinschrift Dashboard.",
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
    "preview_url": "https://render.dinschrift.ch/render/preview/1324567891235646/a1b2c3325e5f6g7h8/STTU169/C001/front"
  }
]
```

### 📦 Garment SKUs
To submit an order, you need the exact SKU for the blank garment and size (e.g., `STSU168C651S` for a size Small, `STSU168C651M` for a Medium). You can view and export a complete list of valid SKUs and their corresponding sizes for your assigned designs directly from the **Shop Settings** page in your Dinschrift Dashboard.

### 2️⃣ Submit an Order
Submit a fulfillment order using the known `design_hash` and standard SKUs. 

**`POST /api/v2/orders`**

**Example Request:**
```json
{
  "external_order_reference": "SHOP-12345",
  "shop_id": "8a7b6c5d",
  "is_test": false,
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
Poll for production status, ready-to-ship dates, and courier tracking details.

**`GET /api/v2/orders/{id}`**

**Example Response:**
```json
{
  "order_id": "101747308500210612",
  "external_order_reference": "SHOP-12345",
  "status": "production",
  "ready_to_ship_date": "2026-07-03",
  "expected_delivery_date": "2026-07-05",
  "tracking": {
    "carrier": "Swiss Post",
    "tracking_number": "99.12.345678.12345678",
    "tracking_url": "[https://service.post.ch/ekp-web/ui/list?ids=99.12.345678.12345678](https://service.post.ch/ekp-web/ui/list?ids=99.12.345678.12345678)"
  },
  "totals": {
    "subtotal": 82.00,
    "grand_total": 88.64,
    "vat_rate": 8.1
  }
}
```

---

### 🪝 Webhooks (Event Push)
Instead of polling the Order Status endpoint, you can configure a Webhook URL in your Dinschrift Dashboard. We will send an HTTP `POST` request to your endpoint whenever an order's status changes. This is the ideal method for integrating with Zapier, Make, or custom backends.

**Supported Events:**
* `order.production_completed` (Item is printed and ready to be sent)
* `order.shipped`
* `order.canceled`

**Security & Verification:**
To ensure webhooks are genuinely from Dinschrift, each request includes an `X-Dinschrift-Signature` header. This signature is an HMAC-SHA256 hash of the raw JSON request body, using your Webhook Secret (available in your dashboard) as the key. 

**Example Webhook Payload (`order.production_completed`):**
```json
{
  "event": "order.production_completed",
  "order_id": "101747308500210612",
  "external_order_reference": "SHOP-12345",
  "timestamp": "2026-06-29T10:45:00Z",
  "data": {
    "status": "ready_to_ship",
    "ready_to_ship_date": "2026-07-03",
    "expected_delivery_date": "2026-07-05"
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
