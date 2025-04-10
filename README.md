# Kong Local Environment Setup
This repository provides a simple setup to run **Kong Gateway** and **Keycloak** on your local machine using **Docker** and **Docker Compose**. It is designed to help you get Kong Gateway running with a Postgres database, and integrate Keycloak for OAuth2-based authentication to secure your APIs.

## Prerequisites

- Docker
- Docker Compose

## Steps to Run the Environment

### 1. Clone the Repository

```bash
git clone <repo-url>
cd Kong-local-env-setup
```
### 2. Start the Environment

- To start all the services, use the following command:
```bash
docker-compose up -d
```
### 3. Configure Keycloak
- Open Keycloak Admin Console: http://localhost:8080
- Log in with the admin credentials: admin / admin
- Create a new realm called kong-realm
- Add a new client with the following details:
  - Client ID: kong-client
  - Root URL: http://localhost:8000/ (Kong's proxy URL)
  - Authorization: Enabled
  - Access Type: Confidential
### 4. Configure Kong
- Add a new service to Kong:
    ```bash
      curl -i -X POST http://localhost:8001/services \
      --data "name=example-service" \
      --data "url=http://example.com"
    ```
- Add a new route to the service:
    ```bash
    curl -i -X POST http://localhost:8001/services/example-service/routes \
    --data "hosts[]=example.com"
  ```
- Enable the OAuth2 plugin on the route:
  ```bash
    curl -i -X POST http://localhost:8001/routes/example-service/plugins \
      --data "name=oauth2" \
      --data "config.client_id=kong-client" \
      --data "config.client_secret=<CLIENT_SECRET>" \
      --data "config.auth_header_name=Authorization" \
      --data "config.scopes=email" \
      --data "config.enable_client_credentials=true"
  ```
### 5. Obtain an OAuth2 Token:
- To get an OAuth2 token using Client Credentials Flow, run the following:
    ```bash
    curl -X POST http://localhost:8080/realms/kong-realm/protocol/openid-connect/token \
    -d "client_id=kong-client" \
    -d "client_secret=<CLIENT_SECRET>" \
    -d "grant_type=client_credentials"
    ```
### 6. Test the API
- Use the access token to make an authenticated request through Kong:
    ```bash
    curl -i -X GET http://localhost:8000/example.com \
    -H "Authorization: Bearer <ACCESS_TOKEN>"
    ```
