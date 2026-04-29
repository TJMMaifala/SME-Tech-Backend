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

All secrets are injected via environment variables. **Never hardcode credentials in `application.yaml`.**

| Variable       | Description                                      | Example                        |
|----------------|--------------------------------------------------|--------------------------------|
| `DB_NAME`      | PostgreSQL username                              | `avnadmin`                     |
| `DB_PASSWORD`  | PostgreSQL password                              | `your-db-password`             |
| `APP_EMAIL`    | Gmail address used for sending emails            | `yourapp@gmail.com`            |
| `APP_PASSWORD` | Gmail App Password (not your account password)   | `xxxx xxxx xxxx xxxx`          |
| `JWT_SECRET`   | Hex-encoded 256-bit secret for JWT signing       | `404E635266556A586E...`        |

### Setting environment variables

**Linux / macOS (bash/zsh):**
```bash
export DB_NAME=avnadmin
export DB_PASSWORD=your-db-password
export APP_EMAIL=yourapp@gmail.com
export APP_PASSWORD="xxxx xxxx xxxx xxxx"
export JWT_SECRET=your-hex-encoded-secret
```

**Windows (PowerShell):**
```powershell
$env:DB_NAME="avnadmin"
$env:DB_PASSWORD="your-db-password"
$env:APP_EMAIL="yourapp@gmail.com"
$env:APP_PASSWORD="xxxx xxxx xxxx xxxx"
$env:JWT_SECRET="your-hex-encoded-secret"
```

**Using a `.env` file (recommended for local dev):**

Create `sme/.env` (already in `.gitignore`):
```
DB_NAME=avnadmin
DB_PASSWORD=your-db-password
APP_EMAIL=yourapp@gmail.com
APP_PASSWORD=xxxx xxxx xxxx xxxx
JWT_SECRET=your-hex-encoded-secret
```

Then load it before running:
```bash
export $(cat .env | xargs)
./mvnw spring-boot:run
```

---

## Database

- **Provider**: Aiven PostgreSQL
- **Host**: `pg-23b34967-sihlentshangase06-6d21.b.aivencloud.com:12667`
- **Database**: `smetech`
- **SSL**: Required (`sslmode=require`)

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
