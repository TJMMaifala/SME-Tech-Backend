# SME Operations Automation System

A multi-tenant SaaS platform for SME owners, automating business registration, online ordering, and customer-facing storefronts.

---

## Prerequisites

- Java 21
- Maven (wrapper included — use `./mvnw`)
- PostgreSQL (Aiven cloud instance — see configuration below)
- Gmail account with an App Password for SMTP

---

## Environment Variables

All secrets are injected via environment variables. **Never hardcode credentials in `application.yaml`.** All variables listed below are **required** — the application will fail to start if any are missing.

| Variable               | Description                                      | Example                        |
|------------------------|--------------------------------------------------|--------------------------------|
| `DB_NAME`              | PostgreSQL username                              | `avnadmin`                     |
| `DB_PASSWORD`          | PostgreSQL password                              | `your-db-password`             |
| `APP_EMAIL`            | Gmail address used for sending emails            | `yourapp@gmail.com`            |
| `APP_PASSWORD`         | Gmail App Password (not your account password)   | `xxxx xxxx xxxx xxxx`          |
| `JWT_SECRET`           | Hex-encoded 256-bit secret for JWT signing       | `404E635266556A586E...`        |
| `APP_BASE_URL`         | Base URL used in email verification links        | `https://yourapp.azurewebsites.net/` |
| `APP_DOMAIN`           | Domain used for public storefront links          | `yourapp.azurewebsites.net`    |
| `CORS_ALLOWED_ORIGINS` | Comma-separated list of allowed CORS origins     | `https://yourapp.netlify.app`  |

### Setting environment variables

**Linux / macOS (bash/zsh):**
```bash
export DB_NAME=avnadmin
export DB_PASSWORD=your-db-password
export APP_EMAIL=yourapp@gmail.com
export APP_PASSWORD="xxxx xxxx xxxx xxxx"
export JWT_SECRET=your-hex-encoded-secret
export APP_BASE_URL="https://sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net/"
export APP_DOMAIN="sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net"
export CORS_ALLOWED_ORIGINS="http://localhost:8080,http://localhost:5173,https://sme-operations.netlify.app,https://sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net/"
```

**Windows (PowerShell):**
```powershell
$env:DB_NAME="avnadmin"
$env:DB_PASSWORD="your-db-password"
$env:APP_EMAIL="yourapp@gmail.com"
$env:APP_PASSWORD="xxxx xxxx xxxx xxxx"
$env:JWT_SECRET="your-hex-encoded-secret"
$env:APP_BASE_URL="https://sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net/"
$env:APP_DOMAIN="sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net"
$env:CORS_ALLOWED_ORIGINS="http://localhost:8080,http://localhost:5173,https://sme-operations.netlify.app,https://sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net/"
```

**Using a `.env` file (recommended for local dev):**

Create `sme/.env` (already in `.gitignore`):
```
DB_NAME=avnadmin
DB_PASSWORD=your-db-password
APP_EMAIL=yourapp@gmail.com
APP_PASSWORD=xxxx xxxx xxxx xxxx
JWT_SECRET=your-hex-encoded-secret
APP_BASE_URL=https://sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net/
APP_DOMAIN=sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net
CORS_ALLOWED_ORIGINS=http://localhost:8080,http://localhost:5173,https://sme-operations.netlify.app,https://sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net/
```

Then load it before running:
```bash
export $(cat .env | xargs)
./mvnw spring-boot:run
```

---

## CORS

CORS is configured via `WebMvcConfig` and applies to all `/api/**` routes.

- **Allowed methods**: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`
- **Allowed headers**: all
- **Credentials**: allowed (cookies / Authorization header)
- **Max preflight cache**: 3600 seconds

Allowed origins are controlled by the `CORS_ALLOWED_ORIGINS` environment variable (comma-separated). There is no default — this variable must be set before starting the application. A typical local dev + production value:

```
http://localhost:8080,http://localhost:5173,https://sme-operations.netlify.app,https://sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net/
```

To add a new origin (e.g. a staging environment), append it to the variable:

```bash
export CORS_ALLOWED_ORIGINS="http://localhost:8080,http://localhost:5173,https://sme-operations.netlify.app,https://sme-operations-dza7e5czhdggexfh.canadacentral-01.azurewebsites.net/,https://staging.yourapp.com"
```

In Azure App Service, update the `CORS_ALLOWED_ORIGINS` application setting in **Configuration → Application Settings**.

---

## Database

- **Provider**: Azure Database for PostgreSQL
- **Host**: `learnwiselydb.postgres.database.azure.com:5432`
- **Database**: `sme_operations_db`
- **SSL**: Required (enforced by Azure)

Schema is managed by Hibernate (`ddl-auto: update`). No manual migrations needed for development.

---

## Running the Application

```bash
# Run (requires environment variables set)
./mvnw spring-boot:run

# Build (skip tests)
./mvnw clean package -DskipTests

# Run tests (uses H2 in-memory DB — no env vars needed)
./mvnw test
```

---

## Deployment

The application is deployed to **Azure App Service** (`sme-operations`) via GitHub Actions on every push to `main`.

### CI/CD Pipeline (`.github/workflows/main_sme-operations.yml`)

| Step | Description |
|------|-------------|
| Build | Compiles and packages the JAR using `mvn clean package -DskipTests` |
| Deploy | Uploads the JAR to Azure App Service (Production slot) |

**Startup command** used by Azure to launch the application:
```
java -jar /home/site/wwwroot/*.jar
```

> Environment variables (DB credentials, JWT secret, mail credentials, CORS origins) must be configured in the Azure App Service **Configuration → Application Settings** panel — they are not bundled into the JAR.

---

## API

Base URL: `http://localhost:8080`  
All endpoints are versioned under `/api/v1/`.

Swagger UI: [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)  
OpenAPI spec: [http://localhost:8080/v3/api-docs](http://localhost:8080/v3/api-docs)

---

## Testing

- Unit tests: JUnit 5 + Mockito
- Property-based tests: jqwik
- Integration tests: H2 in-memory DB (activated via `test` Spring profile)

Tests do **not** require any environment variables — the `test` profile overrides all external dependencies.

```bash
./mvnw test
```
