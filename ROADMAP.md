# Roadmap

## Shipped in v2

| Area | Status |
|---|---|
| Billing, GST split, discounts, split payments, merge bills | ✅ Working |
| Combos and offers (percent/flat, min bill, coupon codes) | ✅ Working |
| Kitchen tickets (KOT) + kitchen board | ✅ Working |
| Table layout and status | ✅ Working |
| Inventory, stock draw-down, reorder status | ✅ Working |
| Suppliers and purchase orders (receive → stock in) | ✅ Working |
| Loyalty points (earn/redeem) | ✅ Working |
| Multi-outlet (data model + assignment) | ✅ Working (basic) |
| GST export (GSTR-1 B2C CSV) | ✅ Working |
| Audit log | ✅ Working |
| Aggregator intake webhook (Swiggy/Zomato) | ✅ Working (middleware contract) |
| Offline mode | ⚠️ App shell cached + offline bill queue; not full sync |
| Nginx + Let's Encrypt auto-renew | ✅ Working (`deploy/install.sh`) |
| Roles: owner/manager/cashier/kitchen | ✅ Working |

## Next

**Near term**
- Thermal printer integration (ESC/POS) for bill and KOT
- Split-bill *by item* (currently split by payment)
- Purchase-order line editor UI (currently prompt-based)
- Full offline sync with conflict resolution and a local queue viewer
- Stock take / wastage entry screen

**Medium term**
- Native Swiggy/Zomato partner API integration (needs partner credentials)
- Customer-facing QR menu and self-order
- Multi-outlet consolidated reporting and stock transfer
- Shift/attendance and payroll export
- Refunds and voids with manager approval

**Longer term**
- GST return filing helpers (GSTR-3B reconciliation)
- Loyalty tiers and campaigns
- Franchise/central kitchen model
- Mobile app for managers (PWA is the first step)
- Webhooks and a public API for third-party integrations

## Known limitations

- Offline mode caches the app and queues bills, but does not yet reconcile
  stock or KOT changes made on the server while offline.
- Aggregator integration is a **contract** (token-protected webhook); it does
  not talk to Swiggy/Zomato directly without partner credentials.
- Multi-outlet is modelled in the data and on orders, but the UI does not yet
  switch between outlets.
- Combos are priced bundles; they do not yet support "choose any 2 of N".
