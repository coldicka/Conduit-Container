# Conduit-Container

This repository contains a legacy full-stack application consisting of a PostgreSQL database, a Django backend, and an Angular frontend. The project is fully containerized using Docker and can be started with Docker Compose.

# Table of Contents

1. [Prerequisites](#Prerequisites) 
2. [Quickstart](#Quickstart) 
3. [Usage](#Usage)
4. [Deployment](#deployment)

## PREREQUISITES

Before you begin, make sure the following software is installed on your system:

- Docker 20.10 or later
- Git
- Python 3.6 or later

## Quickstart

### Clone Repository 

```bash
git clone --recurse-submodules https://github.com/coldicka/Conduit-Container.git
```
### Submodule initialisieren

```bash
git submodule update --init --recursive
```

### Navigate to the project

```bash
cd Conduit-Container
```

### Configure the applications

Rename the provided example configuration files:

```bash
cp example.env .env
```

Next, edit the .env file and configure the required environment variables.

At a minimum, you should set:

* DJANGO_SECRET_KEY
* POSTGRES_PASSWORD
* DJANGO_ALLOWED_HOSTS
* API_BASE_URL
* Complete the "ALLOWED_HOSTS" section by adding your ID at the end.

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

The `.env` file contains all required environment variables.
For Example ... 

| Variable               | Description                                | Example                                 |
| :---: | :---: | :---: |
| DJANGO_SUPERUSER_USERNAME | Your admin password | `test-admin`     |
| DJANGO_SUPERUSER_EMAIL | Your Admin E-mail | `test@test.password` |
| DJANGO_SUPERUSER_PASSWORD | Your admin password | `test-password` |
| POSTGRES_PASSWORD      | PostgreSQL database Passwort               | `Your__Password____`                    |
| DJANGO_ALLOWED_HOSTS   | List of allowed hosts                      |  `localhost,127.0.0.1,backend,YOUR_IP`  |
| PORT                   | Port exposed by the Angular frontend.       | `8282`                                  |
| API_BASE_URL           | Base URL used by the frontend to access the backend API. | `http://YOUR_IP:8282/api` |
| DJANGO_SECRET_KEY      | Django Secret Key                          | zH0c1V5xYxv7LqQ5cFQYq9vP8QJz8JY ... |

### Generating a secret key

The instructions are asking you to generate a random secret key for your Django application and store it in your .env file.

Run this command in your terminal:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

It will output a long, random string, for example:

`zH0c1V5xYxv7LqQ5cFQYq9vP8QJz8JYjQnX7v1hWk2bL6rTmS9Kf0mDgA`

Your output will be different, and that's expected. Each generated key is unique.

Open your .env file and set the DJANGO_SECRET_KEY variable:

`DJANGO_SECRET_KEY=zH0c1V5xYxv7LqQ5cFQYq9vP8QJz8JYjQnX7v1hWk2bL6rTmS9Kf0mDgA`

or, if your .env file uses quotes:

`DJANGO_SECRET_KEY="zH0c1V5xYxv7LqQ5cFQYq9vP8QJz8JYjQnX7v1hWk2bL6rTmS9Kf0mDgA"`

Make sure you use the actual value generated on your machine, not the example above, since the secret key should be unique and kept private.

### Architecture

The application consists of three Docker services:

* Angular Frontend – Provides the web interface.
* Django Backend – Exposes the REST API and contains the application logic.
* PostgreSQL Database – Stores all persistent application data.

The project uses multi-stage Docker builds, which exclude the build environment from the final images. This reduces image size and improves deployment efficiency.

A persistent Docker volume is used for the PostgreSQL database to prevent data loss. All services communicate through an isolated Docker network.

When the backend container starts, the `entrypoint.sh` script automatically applies any pending Django database migrations before launching the application with Gunicorn. Gunicorn is used instead of Django's built-in development server because it is better suited for production environments.

Python dependencies are defined in `requirements.txt`. Files that should not be included in Docker images or committed to the repository are excluded via `.dockerignore` and `.gitignore`.

## Deployment

The repository includes a GitHub Actions workflow located at .github/workflows/deployment.yaml that automatically deploys the application to your server whenever changes are pushed to the main branch.

Before using the deployment workflow, you must configure the required GitHub repository secrets.

### Configure GitHub Secrets

Open your GitHub repository and navigate to:

**Settings** → **Secrets and variables** → **Actions**

![Alternativtext](./images/secrets-and-variables.png)

Create the following repository secrets:

* `SERVER_USER`
* `SERVER_HOST`
* `SERVER_SSH_KEY`
* `DOT_ENV_FILE`

Click New repository secret to create each secret.

![Alternativtext](./images/new-secret.png)

### SERVER_USER
**Name:** `SERVER_USER`
**Secret:** The username used to log in to your deployment server.

### SERVER_HOST
**Name:** `SERVER_HOST`
**Secret:** The IP address or hostname of your deployment server.

### SERVER_SSH_KEY

Create a dedicated SSH key pair for GitHub Actions to access your server.

1. Generate a new SSH key pair on your local machine:

`ssh-keygen -t ed25519 -f ~/.ssh/conduit_pipeline_key`

2. Copy the public key to your server:

`ssh-copy-id -i ~/.ssh/conduit_pipeline_key.pub <user>@<server-ip>`

3. Verify that the public key has been added to the server:

`cat ~/.ssh/authorized_keys`

4. Display the private key:

`cat ~/.ssh/conduit_pipeline_key`

5. Copy the **entire** contents of the private key, including the **-----BEGIN OPENSSH PRIVATE KEY-----** and **-----END OPENSSH PRIVATE KEY-----** lines.

6. In GitHub, create a new repository secret with:

* **Name:** `SERVER_SSH_KEY`
* **Secret:** The copied private key

7. Click Add secret.

> [!IMPORTANT]
> Never share or commit your private SSH key. It should only be stored as a GitHub repository secret.

### DOT_ENV_FILE

Your `.env` file should contain all environment variables required by the application.

Example:

![Alternative text](./images/example-env.png)

1. Fill in all required values.
2. Copy the complete contents of your `.env` file (excluding comments).
3. Create a new repository secret in GitHub.
4. Set the following values:
* **Name:** `DOT_ENV_FILE`
* **Secret:** The complete contents of your .env file
5. Click Add secret.

The deployment workflow will automatically recreate the `.env` file on the server using this secret during deployment.