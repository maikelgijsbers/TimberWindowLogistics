# Copilot Instructions

## Running the app

Open `index.html` directly in a browser. There is no build step, no package manager, and no server required.

## Architecture

The entire application lives in a single file: `index.html`. It contains inline CSS (in `<style>`), HTML markup, and a `<script>` block — no external assets or dependencies.

**Data persistence**: Orders are stored in `localStorage` under the key `timber-window-orders-v1` as a JSON array. `loadOrders()` reads and normalises data; `saveOrders(orders)` writes it back. Every mutation calls `render()` immediately after saving, which re-reads from localStorage and rebuilds the entire DOM.

**Order shape**:
```js
{
  id: string,          // crypto.randomUUID() or fallback timestamp string
  customer: string,
  woodType: "Oak" | "Pine" | "Meranti" | "Accoya",
  width: number,       // mm
  height: number,      // mm
  quantity: number,
  dueDate: string,     // "YYYY-MM-DD"
  street: string,
  houseNumber: string,
  city: string,
  country: string,
  status: string,      // see status constants below
  address?: string,    // legacy field — only on old orders imported without structured address
}
```

**Status lifecycle** (linear, one-way):
`Planned` → `In Production` → `Ready for Delivery` → `Delivered`

Advancement is handled by `nextStatus()`, which steps through the `statuses` array. There is no way to move backwards.

## Key conventions

- **Dates**: Stored as `"YYYY-MM-DD"` ISO strings. Displayed as `"DD/MM/YYYY"` via `formatDate()`. "Today" is always computed via `getTodayLocalDateString()` (local timezone) — never use `new Date().toISOString()` for date comparisons, as it would introduce UTC offset bugs.

- **Metrics**: `total frames` and `in production`/`ready for delivery` counts are sums of `quantity` (not order count). `late orders` is a count of orders (not frames) where `status !== "Delivered"` and `dueDate < today`.

- **Legacy address field**: Old orders may have a top-level `address` string instead of structured fields. The render function handles this with a fallback: `order.address ?? \`${order.street} ${order.houseNumber}, ${order.city}, ${order.country}\``. When editing an order, `address` is explicitly set to `undefined` to migrate it to structured fields.

- **Button event delegation**: Table row actions (Edit, Advance, Remove) use a single `click` listener on `#order-rows`. Buttons carry `data-action` and `data-order-id` attributes; the handler resolves the order by ID from a fresh `loadOrders()` call.

- **Form dual-mode**: The form handles both "new order" and "edit order" via a hidden `editingId` field. `setEditMode(order)` populates the form and shows the Cancel button; `clearEditMode()` resets everything.
