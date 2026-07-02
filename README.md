# Conduit-Container

This repository contains a legacy full-stack application consisting of a PostgreSQL database, a Django backend, and an Angular frontend. The project is fully containerized using Docker and can be started with Docker Compose.

# Table of Contents

1. [Prerequisites](#Prerequisites) 
2. [Quickstart](#Quickstart) 
3. [Usage](#Usage)

## PREREQUISITES

Before you begin, make sure the following software is installed on your system:

- Docker 20.10 or later
- Git
- Python 3.6 or later

## Quickstart

### Clone Repository 
```bash
git clone --recurse-submodules https://github.com/coldicka/Conduit-Container.git &&
```

### Navigate to the project

```bash
cd Conduit-Container
 ```

 ### Configure the application

Rename the provided example configuration files:
```bash
mv example.env .env

cd frontend/src/app/core/interceptors
mv example.api.config.ts api.config.ts # rename the example.api.config.ts to api.config.ts
```

Next, edit the .env file and configure the required environment variables.

At a minimum, you should set:

* DJANGO_SECRET_KEY
* POSTGRES_PASSWORD
* DJANGO_ALLOWED_HOSTS
* API_BASE_URL

Generate a secure Django secret key with:

```bash
python -c "import secrets; print(secrets.token_urlsafe(50))"
```

### Build the Docker images

```bash
cd Conduit-Container/
docker compose build
```  
### Start the application
```bash
docker compose up -d
```

### Open the application

```bash
http://<HOST_IP>:8282
```

### Open the API

```bash
http://<HOST_IP>:8282/api
```

For example:
```bash
http://<HOST_IP>:8282/api/articles
```

### Open the admin

```bash
http://<HOST_IP>:8000/admin
```

## Usage
The .env file contains all required environment variables.
For Example ... 

| Variable               | Description                                | Example                                 |
| :---: | :---: | :---: |
| DJANGO_SECRET_KEY      | Secret key used by Django                  | `g9!Qv4$KL2..`                          |
| POSTGRES_PASSWORD      | PostgreSQL database Passwort               | `Your__Password____`                    |
| DJANGO_ALLOWED_HOSTS   | List of allowed hosts                      |  `localhost,127.0.0.1,backend,YOUR_IP`  |
| PORT                   | ort exposed by the Angular frontend.       | `8282`                                  |
| API_BASE_URL           | Base URL used by the frontend to access the backend API. | `http://YOUR_IP:8282/api` |


### Architecture

The application consists of three Docker services:

* Angular Frontend – Provides the web interface.
* Django Backend – Exposes the REST API and contains the application logic.
* PostgreSQL Database – Stores all persistent application data.

To improve security, the backend is not exposed directly to the internet. Instead, all incoming requests are routed through an Nginx reverse proxy.

The project uses multi-stage Docker builds, which exclude the build environment from the final images. This reduces image size and improves deployment efficiency.

A persistent Docker volume is used for the PostgreSQL database to prevent data loss. All services communicate through an isolated Docker network.

When the backend container starts, the entrypoint.sh script automatically applies any pending Django database migrations before launching the application with Gunicorn. Gunicorn is used instead of Django's built-in development server because it is better suited for production environments.

Python dependencies are defined in requirements.txt. Files that should not be included in Docker images or committed to the repository are excluded via `.dockerignore` and `.gitignore`.
