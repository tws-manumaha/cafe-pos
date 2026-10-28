# Cafe POS

An open-source **restaurant POS and management platform** for cafés and small
restaurants — billing, kitchen, inventory, purchasing, loyalty, offers and
profit reporting in one application.

Built with **Python (FastAPI) + SQLAlchemy**, so it runs against **SQLite**,
**PostgreSQL** or **Microsoft SQL Server** by changing one environment variable.
The whole app is served over HTTPS by **Nginx with automatic Let's Encrypt
certificate renewal**.

> The recipe/costing model (409 ingredients, 681 recipe lines, 99 dishes) was
> designed in the companion Excel workbook and is imported at seed time.

---

## Features

**Billing & front of house**
- Fast dish grid with search and category filter, combo buttons
- Discounts (percent / flat), offer codes, GST split out correctly
- **Split payments** (cash + UPI + card on one bill) with validation
- **Merge bills**, table assignment, dine-in / takeaway / aggregator channels
- Printable receipt

**Kitchen**
- **KOT** — a kitchen ticket is created automatically per order
- Kitchen board with Pending → Preparing → Ready → Served

**Menu & costing**
- Per-dish ingredient cost, price and margin, from the recipe
- Automatic **stock draw-down** on every sale

**Inventory & purchasing**
- Live stock, reorder levels, Out / Reorder / Low / OK status
- **Suppliers** and **purchase orders**; receiving a PO increases stock and
  refreshes the ingredient cost

**Marketing**
- **Offers**: percent or flat, minimum bill, optional coupon code
- **Combos**: bundle dishes at a fixed price
- **Loyalty**: customers earn points per ₹100, redeemable against a bill

**Reporting**
- Today's dashboard, item performance ranking, last 14 days
- **P&L**: revenue − COGS − overheads − salaries = net profit
- **GSTR-1 CSV export** (B2C summary, HSN 9963, CGST/SGST split)
- **Audit log** of every sensitive action

**Platform**
- Multi-user with roles: **owner / manager / cashier / kitchen**
- JWT auth, PBKDF2-SHA256 password hashing, role checks on every write
- **Multi-outlet** and table layout
- **Offline mode** — the app shell is cached by a service worker and bills
  taken while offline are queued on the device and synced when the network returns
- Aggregator intake endpoint for **Swiggy / Zomato** middleware (token-protected)

---

## Quick start (local)

```bash
cd cafe_pos
python3 -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m app.seed
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Open <http://127.0.0.1:8000>. Demo logins:

| User | Password | Role |
|---|---|---|
| owner | owner123 | owner |
| manager | manager123 | manager |
| cashier | cashier123 | cashier |
| kitchen | kitchen123 | kitchen |

**Change these before going live.**

`./run.sh` (Linux/macOS) or `run.bat` (Windows) does the above in one step.

---

## Configuration

All via environment variables (see `deploy/.env.example`):

| Variable | Default | Notes |
|---|---|---|
| `DATABASE_URL` | `sqlite:///./cafe.db` | Postgres / MSSQL examples in `INSTALL.md` |
| `JWT_SECRET` | dev value | **Set a long random string in production** |
| `JWT_EXPIRE_MIN` | `720` | Token lifetime |
| `GST_RATE` | `0.05` | |
| `WASTE_RATE` | `0.08` | Wastage added to every recipe cost |
| `AGGREGATOR_TOKEN` | dev value | Required header `X-AGG-TOKEN` for aggregator intake |

---

## Production deployment (Nginx + Let's Encrypt)

On a fresh Debian/Ubuntu server:

```bash
sudo DOMAIN=cafe.example.com EMAIL=you@example.com ./deploy/install.sh
```

The script installs Python, PostgreSQL, Nginx and Certbot, creates a service
user, builds the virtualenv, seeds the database, installs a **systemd** unit
(`deploy/cafe-pos.service`), configures the **Nginx reverse proxy**
(`deploy/nginx-cafe-pos.conf`) and obtains a **Let's Encrypt** certificate with
**automatic renewal** via `certbot.timer`.

Full manual steps, Windows/IIS and Docker notes are in **[INSTALL.md](INSTALL.md)**.

---

## Project layout

```
app/
  db.py          engine/session — one DATABASE_URL for SQLite/Postgres/MSSQL
  models.py      SQLAlchemy models (users, dishes, recipes, orders, KOT, POs…)
  security.py    PBKDF2 password hashing + JWT
  main.py        FastAPI app and all REST endpoints
  seed.py        loads data/app_data.json (from the Excel workbook)
static/          single-page front end (no build step) + service worker
deploy/          nginx, systemd, .env example, install.sh
data/            app_data.json (ingredients, dishes, recipes)
```

API docs are generated automatically at **`/docs`** (Swagger UI).

---

## License

MIT — see [LICENSE](LICENSE).

## Roadmap

See [ROADMAP.md](ROADMAP.md).
