# Django CI/CD Project - Docker Setup

This project uses Docker for production deployment and CI/CD pipelines. For local development, run Django natively without Docker.

## Development Workflow

For local development, run Django directly:

```bash
# Install dependencies
pip install -r requirements.txt

# Run migrations
python manage.py migrate

# Start development server
python manage.py runserver
```

This approach provides faster iteration, easier debugging, and native IDE integration.

## Production Deployment with Docker

The `Dockerfile` creates a production-ready image that:
- Uses Python 3.13-slim base image
- Installs all dependencies from `requirements.txt`
- Runs the application with Gunicorn WSGI server (3 workers)
- Uses a non-root user for security
- Exposes port 8000

### Building the Docker Image

```bash
# Build the image
docker build -t django-cicd:latest .

# Tag for your registry (example)
docker tag django-cicd:latest yourusername/django-cicd:latest
docker tag django-cicd:latest yourusername/django-cicd:v1.0.0
```

### Running in Production

```bash
# Run the container with production settings
docker run -d \
  -p 8000:8000 \
  -e DEBUG=False \
  -e SECRET_KEY=your-production-secret-key \
  -e ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com \
  --name django-app \
  django-cicd:latest
```

### Environment Variables for Production

**Required environment variables:**

- `DEBUG` - **Must be `False` in production**
- `SECRET_KEY` - Django secret key (generate a secure one)
- `ALLOWED_HOSTS` - Comma-separated list of allowed domain names

**Generate a secure SECRET_KEY:**
```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

### Using docker-compose.yml for Testing

The `docker-compose.yml` is provided for local testing that mirrors production:

```bash
# Test the production setup locally
docker-compose up
```

**Note:** This runs Gunicorn, not the Django dev server, so it's useful for testing production-like behavior.

### Example: Deploy to Production

```bash
# Pull and run the latest image on production server
docker pull myapp:latest
docker stop django-app || true
docker rm django-app || true
docker run -d \
  -p 8000:8000 \
  --env-file /path/to/production.env \
  --name django-app \
  --restart unless-stopped \
  myapp:latest
```

## Useful Commands

```bash
# View logs from running container
docker logs -f django-app

# Execute management commands in running container
docker exec django-app python manage.py migrate
docker exec django-app python manage.py createsuperuser
docker exec django-app python manage.py collectstatic --noinput

# Stop and remove container
docker stop django-app
docker rm django-app

# Remove old images to save space
docker image prune -a
```

## Project Structure

```
.
├── Dockerfile              # Production image definition
├── docker-compose.yml      # For local production testing
├── .dockerignore          # Files excluded from Docker build
├── .env.example           # Example environment variables
├── requirements.txt       # Python dependencies
└── djangoCICD/            # Django project
```

## Troubleshooting

### Container won't start
Check logs: `docker logs django-app`

### Port already in use
Change the port mapping: `-p 8080:8000` (maps container's 8000 to host's 8080)

### Permission issues
Ensure proper file permissions or adjust the USER in Dockerfile if needed

