# Django CI/CD Project

## Project Purpose

This is a **learning project** designed to practice and demonstrate:
- **Continuous Integration & Continuous Deployment (CI/CD)** using GitHub Actions
- **Containerization** with Docker
- **Automated Deployment** to production servers
- **Production-ready** Django application setup

## Learning Goals

This project showcases:

1. **GitHub Actions Workflows** - Automated testing, building, and deployment pipelines
2. **Docker & Containerization** - Creating production-ready Docker images
3. **GHCR (GitHub Container Registry)** - Pushing images to a container registry
4. **Remote Server Deployment** - Automated deployment via SSH to production servers
5. **Environment Configuration** - Managing secrets and environment variables securely
6. **Django Best Practices** - Production-grade Django setup with Gunicorn

## Project Overview

This is a simple Django application that demonstrates a complete CI/CD pipeline:

```
Code Commit (Git) 
    ↓
GitHub Actions Workflow Triggered
    ↓
Build Docker Image
    ↓
Push to GitHub Container Registry (GHCR)
    ↓
Deploy to Production Server (Hetzner) via SSH
    ↓
Running Production Container
```

## How It Works

### Local Development

For local development, run Django natively **without Docker**:

```bash
# Install dependencies
pip install -r requirements.txt

# Run migrations
python manage.py migrate

# Start development server
python manage.py runserver
```

This provides:
- Fast iteration
- Easy debugging
- Native IDE integration

### Production Deployment

When you push to the `main` branch:

1. **GitHub Actions** automatically builds a Docker image
2. Image is tagged and pushed to **GitHub Container Registry (GHCR)**
3. SSH connects to the production server (Hetzner)
4. Pulls the latest Docker image
5. Stops the old container
6. Runs the new container with production settings

## Key Components

### 📁 Project Structure

```
django-ci-cd/
├── .github/
│   └── workflows/
│       └── production-deploy.yml    # CI/CD Pipeline
├── djangoCICD/                      # Django App
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── Dockerfile                       # Production Image Definition
├── requirements.txt                 # Python Dependencies
├── manage.py                        # Django CLI
└── README.md                        # This file
```

### 🐳 Docker

**Dockerfile** creates a production-ready image:
- Python 3.13-slim base image
- All dependencies installed
- Non-root user for security
- Gunicorn WSGI server
- Health-check ready

### 🔄 CI/CD Pipeline (.github/workflows/production-deploy.yml)

**Build & Push:**
- Triggered on push to `main` branch
- Builds Docker image from Dockerfile
- Pushes image to GHCR with `latest` tag

**Deploy:**
- Connects to production server via SSH
- Logs into GHCR (pulls images)
- Stops old container
- Runs new container with production environment variables
- Maps SQLite database volume for persistence
- Auto-restart on failure

## Security Features

✅ Non-root user in Docker container  
✅ GitHub Secrets for sensitive data (SSH keys, API tokens)  
✅ Environment variables for configuration  
✅ Read-only dependencies  
✅ Minimal base image (Python 3.13-slim)

## Technology Stack

- **Language:** Python 3.13
- **Framework:** Django 6.0.2
- **Web Server:** Gunicorn 25.1.0
- **Containerization:** Docker
- **Registry:** GitHub Container Registry (GHCR)
- **CI/CD:** GitHub Actions
- **Hosting:** Hetzner (Production)

## Required Secrets for CI/CD

To enable automated deployment, configure these GitHub Secrets:

- `GHCR_PAT` - GitHub Container Registry Personal Access Token
- `HETZNER_HOST` - Production server IP/hostname
- `HETZNER_USERNAME` - SSH username for production server
- `HETZNER_SSH_KEY` - Private SSH key for production server

## Quick Start

### 1. Clone the Repository
```bash
git clone <your-repo-url>
cd django-ci-cd
```

### 2. Setup Local Development
```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run migrations
python manage.py migrate

# Start development server
python manage.py runserver
```

### 3. Deploy to Production
Just push to `main` branch - GitHub Actions handles the rest!

```bash
git add .
git commit -m "Your changes"
git push origin main
```

## Learn More

This project is perfect for understanding:
- How modern DevOps pipelines work
- Docker basics and best practices
- GitHub Actions automation
- Production Django deployment
- Container registries and image management
- Infrastructure as Code (IaC) concepts

## Notes

- This is a **learning project**, not a production-ready application
- Modify the GitHub Actions workflow for your own server/registry
- Always use strong, unique `SECRET_KEY` and `DEBUG=False` in production
- Use PostgreSQL instead of SQLite for production databases
- Implement proper logging, monitoring, and error handling for production use

---

Created for learning CI/CD, automated deployment, and GitHub Actions automation.

