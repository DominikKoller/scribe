# Scribe

This repository contains the source code for [https://scribe.dominikkoller.com](https://scribe.dominikkoller.com). It is composed of two submodules:

- A **Backend** including a Traefik Load Balancer, MongoDB Database and a Node.js Apollo + Hocuspocus server
- A **SvelteKit frontend**

This repository includes all the necessary information to run and deploy both the backend and the frontend. Traefik handles SSL termination and obtains and renews SSL certificates from Let's Encrypt.

Certificates are stored automatically in the folder `/letsencrypt`.

## Components

### Backend

- **Language:** Typescript
- **Framework:** Node.js
- **Database:** MongoDB
- **Environment Variables:** Located in `backend/.env`

### Frontend
- **Language:** Typescript
- **Framework:** SvelteKit
- **Environment Variables:** Located in `frontend/.env`

## Prerequisites

- Docker and Docker Compose installed on your system.
- For local deployment:
    - OpenSSL to generate local certificates.
    - Ability to modify your `/etc/hosts` file.

## Setup Instructions

### Certificates Storage

Certificates are automatically stored in the `/letsencrypt` folder by Traefik.

### Setting up Traefik Dashboard Authentication

Before running the application, you should set up a hashed password file for the Traefik Dashboard:

```bash
echo "admin:$(openssl passwd -apr1 strongpassword)" > traefik-dashboard-auth/.htpasswd
```

Replace `strongpassword` with a strong password of your choice. This command stores the login information used by Traefik in `traefik-dashboard-auth/.htpasswd`.

### MongoDB Express Configuration

For MongoDB Express, you must set the following in `.env.mongo`:

```env
MONGO_INITDB_ROOT_USERNAME=root
MONGO_INITDB_ROOT_PASSWORD=password
ME_CONFIG_MONGODB_ADMINUSERNAME=root
ME_CONFIG_MONGODB_ADMINPASSWORD=password
ME_CONFIG_MONGODB_URL=mongodb://root:password@mongodb:27017
ME_CONFIG_BASICAUTH_USERNAME=admin
ME_CONFIG_BASICAUTH_PASSWORD=password
```

Make sure to replace `root`, `password`, and `admin` with secure usernames and passwords.

### Backend Environment Variables

The same MongoDB username and password must be set in `backend/.env` (note: the `BASICAUTH` variables are for access to MongoDB Express and are not needed here):

```env
MONGO_USERNAME=root
MONGO_PASSWORD=password
```

Additionally, the backend `.env` must contain:

```env
EXPRESS_PORT=3001
HOCUSPOCUS_PORT=3002
JWT_SECRET=jwt_secret  # Replace with your JWT secret
OPENAI_API_KEY=openaikey
```

Replace `jwt_secret`, and `openaikey` with your actual JWT secret, and OpenAI API key, respectively.

### Frontend Environment Variables

The frontend `.env` must contain the following variables. For local development, use the provided values. In production, use the actual domain:

```env
PUBLIC_API_BASE_URL=https://scribe.localhost/api/
PUBLIC_APOLLO_URL=https://scribe.localhost/graphql
PUBLIC_HOCUSPOCUS_URL=https://scribe.localhost/hocuspocus
```

## Deployment

### Production Deployment

To deploy the application in production, run:

```bash
docker-compose up --build -d
```

If you choose to enable it, the Traefik dashboard will be reachable via `https://scribe.dominikkoller.com:8080` (assuming that is the domain you deploy to). If you are deploying to a different domain, you will need to change the `docker-compose.yml` file accordingly (which is also where you can enable or disable the dashboard)

**Note:** Make sure your port `8080` is opened, e.g., in your AWS EC2 instance configuration.

### Local Deployment

To deploy locally, run:

```bash
docker-compose -f docker-compose.local.yml up --build
```

#### Generating Local Certificates

For your local HTTPS connection, you need to create local certificates:

```bash
mkdir certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/privkey.pem -out certs/fullchain.pem \
  -subj "/CN=scribe.localhost"
```

#### Updating Hosts File

Add the following entry to your `/etc/hosts` file:

```
127.0.0.1 scribe.localhost
```

#### Note on HTTPS

The services will then be available under `https://scribe.localhost/`. Make sure to type `https`, as before you have accepted your local certificate, your browser might not allow an automatic redirect to `https`.

**When you get a 404 error, check that your browser is going to `https`!**

## Notes

### SvelteKit Frontend Communication

The SvelteKit frontend's browser communicates directly with the backend. This decouples the SvelteKit application from the backend—it can be hosted independently or replaced with any other client that has no hosting server.

### Docker Containers Hosting

These Docker containers are currently hosted on an AWS EC2 instance.
