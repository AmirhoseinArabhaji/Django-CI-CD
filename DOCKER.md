# Django CI/CD Project - Docker Setup

This project uses **PostgreSQL** as the database (instead of SQLite) and provides two Docker Compose configurations:

- **`docker-compose.yml`** — Production/deployment (PostgreSQL + Django app via Gunicorn)
- **`docker-compose.dev.yml`** — Development (PostgreSQL only; run Django natively)

All PostgreSQL credentials and Django settings are managed through environment variables in a `.env` file.

---

## Environment Variables

Copy the example file and edit it:

```bash
cp .env.example .env
```

| Variable                | Description                              | Example                              |
|-------------------------|------------------------------------------|--------------------------------------|
| `DEBUG`                 | Django debug mode                        | `True` (dev) / `False` (prod)        |
| `SECRET_KEY`            | Django secret key                        | *(generate a secure one for prod)*   |
| `ALLOWED_HOSTS`         | Comma-separated allowed hosts            | `localhost,127.0.0.1`                |
| `POSTGRES_DB`           | PostgreSQL database name                 | `postgres_db`                        |
| `POSTGRES_USER`         | PostgreSQL user                          | `postgres_user`                      |
| `POSTGRES_PASSWORD`     | PostgreSQL password                      | `postgres_password`                  |
| `POSTGRES_HOST`         | PostgreSQL host                          | `localhost` (dev) / `db` (prod)      |
| `POSTGRES_PORT`         | PostgreSQL port                          | `5432`                               |
| `CSRF_TRUSTED_ORIGINS`  | Trusted origins for CSRF (prod only)     | `https://yourdomain.com`             |

**Generate a secure `SECRET_KEY`:**

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

> **Important:** In production, `POSTGRES_HOST` must be set to `db` (the Docker Compose service name). For local development, keep it as `localhost`.

---

## Development Workflow

In development, only the PostgreSQL database runs in Docker. You run Django natively for faster iteration, easier debugging, and full IDE integration.

### 1. Start the development database

```bash
docker compose -f docker-compose.dev.yml up -d
```

This starts a PostgreSQL 16 container with:
- Data persisted in a `postgres_data_dev` Docker volume
- Port **5432** exposed on `127.0.0.1` so Django can connect via `localhost`

### 2. Set up your `.env` for development

Make sure your `.env` has:

```dotenv
DEBUG=True
SECRET_KEY=your-dev-secret-key
ALLOWED_HOSTS=localhost,127.0.0.1
POSTGRES_DB=postgres_db
POSTGRES_USER=postgres_user
POSTGRES_PASSWORD=postgres_password
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

### 3. Install dependencies and run Django

```bash
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

### 4. Stop the development database

```bash
docker compose -f docker-compose.dev.yml down
```

To also remove the database volume (deletes all data):

```bash
docker compose -f docker-compose.dev.yml down -v
```

---

## Production Deployment

The production `docker-compose.yml` runs the full stack: **PostgreSQL + Django (Gunicorn)**.

### Architecture

```
docker-compose.yml
├── db      — PostgreSQL 16 (Alpine) with health checks
├── setup   — One-time container: runs migrations & collectstatic, then exits
└── web     — Gunicorn (4 workers) serving Django on port 8000
```

The `setup` service waits for the database to be healthy, runs `migrate` and `collectstatic`, then exits. The `web` service waits for `setup` to complete successfully before starting.

### 1. Set up your `.env` for production

```dotenv
DEBUG=False
SECRET_KEY=your-production-secret-key
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
POSTGRES_DB=postgres_db
POSTGRES_USER=postgres_user
POSTGRES_PASSWORD=a-strong-random-password
POSTGRES_HOST=db
POSTGRES_PORT=5432
CSRF_TRUSTED_ORIGINS=https://yourdomain.com,https://www.yourdomain.com
```

> **Note:** `POSTGRES_HOST=db` is required in production — it refers to the Docker Compose service name.

### 2. Start the production stack

```bash
docker compose up -d
```

This will:
1. Start PostgreSQL and wait for it to be healthy
2. Run migrations and collect static files (via the `setup` service)
3. Start Gunicorn on `127.0.0.1:8000`

### 3. Check status and logs

```bash
# View all services
docker compose ps

# Follow logs from all services
docker compose logs -f

# Follow logs from a specific service
docker compose logs -f web
```

### 4. Stop the production stack

```bash
docker compose down
```

To also remove the database volume (deletes all data):

```bash
docker compose down -v
```

---

## Dockerfile

The `Dockerfile` creates a production-ready image that:

- Uses **Python 3.13-slim** base image
- Installs system dependencies (`libpq-dev` for PostgreSQL)
- Installs all Python dependencies from `requirements.txt`
- Runs the application with **Gunicorn** (4 workers)
- Uses a **non-root user** (`django`) for security
- Exposes port **8000**

### Building the Docker image manually

```bash
docker build -t django-cicd:latest .
```

---

## Useful Commands

```bash
# Run Django management commands in the production container
docker compose exec web python manage.py createsuperuser
docker compose exec web python manage.py shell
docker compose exec web python manage.py showmigrations

# Connect to the PostgreSQL database
docker compose exec db psql -U ${POSTGRES_USER} -d ${POSTGRES_DB}

# Restart only the web service
docker compose restart web

# Rebuild and restart everything
docker compose up -d --build
```

---

## Project Structure

```
.
├── Dockerfile              # Production image definition
├── docker-compose.yml      # Production: PostgreSQL + Django (Gunicorn)
├── docker-compose.dev.yml  # Development: PostgreSQL only
├── .env.example            # Example environment variables
├── .env                    # Your local environment variables (not committed)
├── requirements.txt        # Python dependencies
├── manage.py               # Django management script
└── djangoCICD/             # Django project
    ├── settings.py         # Settings (reads from env vars)
    ├── urls.py
    ├── wsgi.py
    └── asgi.py
```

---

## Troubleshooting

### Database connection refused
- **Development:** Make sure the dev database is running: `docker compose -f docker-compose.dev.yml up -d`
- **Production:** Ensure `POSTGRES_HOST=db` in your `.env`
- Check if PostgreSQL is healthy: `docker compose ps`

### Container won't start
Check logs: `docker compose logs web`

### Port already in use
- **Web (8000):** Change the port mapping in `docker-compose.yml`: `"127.0.0.1:8080:8000"`
- **PostgreSQL (5432):** Change the port mapping in `docker-compose.dev.yml`: `"127.0.0.1:5433:5432"` and update `POSTGRES_PORT=5433` in `.env`

### Migrations failed
Check the setup service logs: `docker compose logs setup`

### Permission issues
The Dockerfile uses a non-root `django` user (UID 1000). Ensure mounted volumes have compatible permissions.

