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
```
