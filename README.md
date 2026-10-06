# minhajvault

A concise Spring Boot REST API for managing users. Built with Spring Boot 4.1.x, Spring Data JPA, and PostgreSQL (H2 is used for tests).

## Quickstart

Prerequisites:
- Java 21
- Maven
- PostgreSQL (or let tests use H2)

Build and run locally:

```bash
mvn clean package
mvn spring-boot:run
```

The app listens on http://localhost:8080 by default.

Run tests:

```bash
mvn test
```

## Configuration

- Main config: [src/main/resources/application.properties](src/main/resources/application.properties#L1)
- Test config: [src/test/resources/application.properties](src/test/resources/application.properties#L1)

Tests run against an in-memory H2 DB and are configured to disable Vault so `mvn test` works without external services.

## Vault integration

This project can retrieve PostgreSQL credentials from HashiCorp Vault (PostgreSQL secrets engine or KV). When using Vault, do not hardcode `spring.datasource.username` or `spring.datasource.password` in `src/main/resources/application.properties`.

Example minimal config in `application.properties` when using Vault:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/minhajvault
spring.cloud.vault.postgresql.enabled=true
```

Environment variables for a local AppRole-based Vault setup (example):

```bash
export VAULT_URI=http://127.0.0.1:8200
export VAULT_AUTH_METHOD=APPROLE
export VAULT_ROLE_ID=<role_id>
export VAULT_SECRET_ID=<secret_id>
export VAULT_KV_ENABLED=true
export VAULT_KV_BACKEND=secret
export VAULT_KV_DEFAULT_CONTEXT=test/pg
export VAULT_KV_APPLICATION_NAME=test/pg
```

The application will fetch DB credentials from Vault at startup when configured to do so.

If you want to run a local Vault dev server for testing, see the `vault-config/` and `vault-data/` folders included in the repo for example configuration and data snapshots.

## Local PostgreSQL (optional)

Create a DB and user for local runs (example):

```bash
sudo -u postgres psql -c "CREATE USER minhaj WITH PASSWORD '....';"
sudo -u postgres psql -c "CREATE DATABASE minhajvault OWNER minhaj;"
```

## API Endpoints

- GET /api/users
- GET /api/users/{id}
- GET /api/users/by-email?email={email}
- POST /api/users
- PUT /api/users/{id}
- DELETE /api/users/{id}

Example create user:

```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Minhaj","email":"minhaj@example.com"}'
```

## Useful files

- [src/main/java](src/main/java#L1) — application sources
- [src/test/java](src/test/java#L1) — tests
- [vault-config/vault.hcl](vault-config/vault.hcl#L1) — example Vault server config

## Notes

- `mvn test` should pass without external services.
- For Vault AppRole values, a file `role id of vault.txt` is included in the repo (handle secrets carefully).

Add a small `docker-compose` for PostgreSQL + Vault to simplify local testing.
