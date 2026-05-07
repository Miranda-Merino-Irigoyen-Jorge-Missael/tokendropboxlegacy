# AccessTokenDropbox

## Overview

AccessTokenDropbox is a microservice built with FastAPI, designed to securely manage, retrieve, and automatically refresh Dropbox OAuth2 access tokens. By integrating directly with Google Cloud Secret Manager, it acts as a centralized and secure broker for other applications within an infrastructure that require interacting with the Dropbox API, abstracting away the complexity of token lifecycle management.

## Architecture

*   **Framework:** FastAPI (Python)
*   **Storage Backend:** Google Cloud Secret Manager
*   **Authentication:** Signature-based validation (`API_SECRET_KEY`)
*   **Server:** Uvicorn

## Prerequisites

To run this service, the following requirements must be met:

1.  Python 3.9 or higher.
2.  A Google Cloud Project with the **Secret Manager API** enabled.
3.  Appropriate Google Cloud Service Account credentials with permissions to read, write, and create versions in Secret Manager.
4.  A registered Dropbox application with an `App Key` and `App Secret`.

## Configuration

The application is configured via environment variables. In a local environment, these can be provided using a `.env` file in the root directory.

| Variable | Description | Example |
| :--- | :--- | :--- |
| `APP_NAME` | Name of the application instance. | `AccessTokenDropbox` |
| `GCP_PROJECT_ID` | The ID of the Google Cloud Project hosting the secrets. | `your-gcp-project-id` |
| `SECRET_NAME` | The identifier for the secret in Secret Manager. | `dropbox-access-token` |
| `API_SECRET_KEY` | The pre-shared key used to authenticate incoming requests. | `your-secure-key` |
| `DROPBOX_APP_KEY` | The App Key provided by the Dropbox Developer Console. | `dropbox-key` |
| `DROPBOX_APP_SECRET` | The App Secret provided by the Dropbox Developer Console. | `dropbox-secret` |

## Local Setup

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/Miranda-Merino-Irigoyen-Jorge-Missael/tokendropboxlegacy.git
    cd tokendropboxlegacy
    ```

2.  **Create and activate a virtual environment:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows: .\venv\Scripts\activate
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Set up environment variables:**
    Create a `.env` file based on the configuration table above. Ensure that your Google Cloud authentication is properly configured in your terminal session (e.g., via `gcloud auth application-default login` or setting the `GOOGLE_APPLICATION_CREDENTIALS` environment variable).

5.  **Run the application:**
    ```bash
    uvicorn app.main:app --host 0.0.0.0 --port 8080 --reload
    ```

## API Reference

The service exposes several endpoints under the `/api/v1` prefix. All token operations require authentication via a JSON payload containing the `signature` that matches the `API_SECRET_KEY`.

### Retrieve Token
Retrieves a valid Dropbox access token. If the current token has expired, the service will automatically request a new one using the refresh token before responding.

**Request:**
```http
POST /api/v1/token
Content-Type: application/json

{
    "signature": "your-api-secret-key"
}
```

**Response (Success):**
```json
{
    "access_token": "sl.xxxxxxxxxxxxxxx",
    "expires_in": 14400,
    "refreshed": false
}
```

### Initialize Token
Initializes the token for the first time using an OAuth2 authorization code obtained from Dropbox.

**Request:**
```http
POST /api/v1/token/initialize?signature=your-api-secret-key&authorization_code=dropbox-auth-code
```

### Force Refresh
Forces the generation of a new access token, bypassing any expiration checks.

**Request:**
```http
POST /api/v1/token/refresh?signature=your-api-secret-key
```

### Health Check
Endpoint used for load balancers and container orchestration to verify the service status.

**Request:**
```http
GET /health
```
**Response:**
```json
{
    "status": "healthy",
    "service": "AccessTokenDropbox",
    "version": "1.0.0"
}
```
