# Key System

A robust and flexible key system management application built with Go, Iris web framework, and MongoDB. It allows you to manage software licenses (referred to as "apps") and provides a gateway for users to activate their software by completing a series of advertisement-linked tasks.

## Features

- **App Management**: Create, edit, and delete applications with custom license parameters.
- **Dynamic Gateway**: Configurable advertisement links that users must visit before activation.
- **Identifier Tracking**: Activations are tied to a specific hardware identifier.
- **Activation Expiration**: Set custom expiration hours for each application (defaults to 16 hours).
- **IP Rate Limiting**: Limits an IP to 5 activations per rolling 24-hour period to prevent abuse.
- **Captcha Verification**: Integration with Cloudflare Turnstile for secure human verification.
- **Database Support**: Primary support for MongoDB
- **Metrics**: Integrated Prometheus metrics endpoint (`/metrics`) for monitoring.
- **Dockerized**: Easy deployment using Docker.
- **Dynamic Templates**: Loads templates from `/data/templates` if it exists, otherwise defaults to the local `templates/` directory.

## Environment Variables

The application is configured using the following environment variables. A `.env.example` file is provided as a reference.

| Variable | Description |
|----------|-------------|
| `CONNECTION_STRING` | MongoDB connection URI. |
| `DB_NAME` | The name of the MongoDB database. |
| `LICENSE_COLLECTION` | Collection name for storing application licenses. |
| `ACTIVATION_COLLECTION` | Collection name for storing active user activations. |
| `ACTIVATION_LOG_COLLECTION` | Collection name for logging activation events (used for rate limiting). |
| `CAPTCHA_SECRET` | Your Cloudflare Turnstile secret key. |
| `CAPTCHA_SITEKEY` | Your Cloudflare Turnstile site key. |
| `ADMIN_USERNAME` | Admin username for the admin panel. |
| `ADMIN_PASSWORD` | Admin password for the admin panel. |
| `ENCRYPTION_KEY` | Secret key used for XOR encryption/decryption of cookies. |
| `LOKI_URL` | Grafana Loki push API endpoint. |
| `LOOTLABS_KEY` | API key for Lootlabs integration. |
| `DOMAIN` | Domain url for redirect creation with lootlabs. |

## Installation & Setup

### Prerequisites

- MongoDB instance (or compatible)
- Cloudflare Turnstile account (for captcha)

### Docker Compose

This is the easiest way to run the application along with a local MongoDB instance.

1.  **Configure environment variables**:
    Copy `.env.example` to `.env` and set `CONNECTION_STRING=mongodb://mongodb:27017` and other values as needed.

2.  **Start with Docker Compose**:
    ```bash
    docker-compose up -d
    ```
    This will build the application image and start it along with a MongoDB container. The application will wait for MongoDB to be healthy before starting.

3.  **Check logs**:
    ```bash
    docker-compose logs -f keysystem
    ```

## API Usage

### User Activation Gateway

Users access the gateway via:
`GET /gateway?app_id={appId}&identifier={identifier}`

- They will be redirected through the configured `advertLinks` for the app.
- After completing the links, they solve a captcha and the activation is stored.

### Client Check (Is Keyed?)

Clients can check their activation status:
`GET /api/check?app_id={appId}&identifier={identifier}`

- **Success (200)**: Returns JSON indicating success and expiration time.
- **Failure (401)**: Returns JSON indicating the failure for expired identifier
- **Failure (404)**: Returns JSON indicating the failure for no identifier saved

### Admin Panel

The admin panel is located at `/apps/manage`. It requires login using the credentials defined in `ADMIN_USERNAME` and `ADMIN_PASSWORD`.

## Monitoring

Prometheus metrics are available at `/metrics`. You can monitor:
- Success and failure counts for key checks.
- Captcha verification failures.
- Rate limit hits.
- General application health.