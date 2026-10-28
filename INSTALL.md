# Installation

## 1. Local / single machine

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m app.seed
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Other devices on the same network can use `http://<your-ip>:8000`.

## 2. Choosing a database

Set `DATABASE_URL` before starting:

```bash
# SQLite — zero setup, fine for one till
export DATABASE_URL="sqlite:///./cafe.db"

# PostgreSQL
export DATABASE_URL="postgresql+psycopg2://cafepos:password@localhost:5432/cafepos"
pip install psycopg2-binary

# Microsoft SQL Server
export DATABASE_URL="mssql+pyodbc://sa:password@localhost/cafepos?driver=ODBC+Driver+17+for+SQL+Server"
pip install pyodbc
```

Tables are created automatically on first start.

## 3. Production on Debian/Ubuntu (Nginx + Let's Encrypt)

```bash
sudo DOMAIN=cafe.example.com EMAIL=you@example.com ./deploy/install.sh
```

What it does:
1. Installs `python3-venv`, `nginx`, `certbot`, `postgresql`
2. Creates the `cafepos` system user and copies the app to `/opt/cafe-pos`
3. Builds the virtualenv and installs requirements
4. Creates `/opt/cafe-pos/.env` with a random `JWT_SECRET`
5. Creates the `cafepos` PostgreSQL database and role
6. Seeds the database
7. Installs and starts `cafe-pos.service` (systemd)
8. Configures Nginx as a reverse proxy to `127.0.0.1:8000`
9. Runs Certbot for HTTPS and enables `certbot.timer` for **automatic renewal**

### Manual steps (if you prefer)

```bash
sudo cp deploy/cafe-pos.service /etc/systemd/system/
sudo systemctl daemon-reload && sudo systemctl enable --now cafe-pos

sudo cp deploy/nginx-cafe-pos.conf /etc/nginx/sites-available/cafe-pos
sudo ln -s /etc/nginx/sites-available/cafe-pos /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

sudo certbot --nginx -d cafe.example.com --agree-tos -m you@example.com --redirect
sudo systemctl enable --now certbot.timer      # auto-renew
sudo certbot renew --dry-run                   # verify renewal works
```

## 4. Windows / IIS

Python apps can't run inside IIS natively. The supported pattern:

1. Run `uvicorn app.main:app --host 127.0.0.1 --port 8000` as a Windows service
   (e.g. with [NSSM](https://nssm.cc/)).
2. Install **IIS** with **Application Request Routing (ARR)** + URL Rewrite.
3. Create a reverse-proxy rule from your site to `http://127.0.0.1:8000`.

## 5. Docker (optional)

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
docker build -t cafe-pos .
docker run -d -p 8000:8000 -v cafepos_data:/app/data \
  -e JWT_SECRET=$(openssl rand -hex 32) --name cafe-pos cafe-pos
```

## 6. Post-install checklist

- [ ] Change the four demo passwords (or delete the demo users)
- [ ] Set a strong `JWT_SECRET`
- [ ] Confirm HTTPS works and `certbot renew --dry-run` passes
- [ ] Point `DATABASE_URL` at PostgreSQL/MSSQL for anything beyond a single till
- [ ] Set up nightly database backups (`pg_dump`)
- [ ] Set `AGGREGATOR_TOKEN` before enabling Swiggy/Zomato intake
