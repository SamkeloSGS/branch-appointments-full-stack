# Branch Appointments Fullstack

Runs the whole Branch Appointments app from its Docker images: the frontend and backend, plus PostgreSQL, Redis,
Vault, Keycloak, Mailpit and monitoring. No source code lives here.

The images come from the other two projects, whose GitHub workflows build them and push them to Docker Hub:

| Image | Project | Workflow |
|---|---|---|
| [`samkelo1999/branch-appointments-backend`](https://hub.docker.com/r/samkelo1999/branch-appointments-backend) | [branch-appointments-backend](https://github.com/SamkeloSGS/branch-appointments-backend) | `.github/workflows/backend-image.yml` |
| [`samkelo1999/branch-appointments-frontend`](https://hub.docker.com/r/samkelo1999/branch-appointments-frontend) | [branch-appointments-frontend](https://github.com/SamkeloSGS/branch-appointments-frontend) | `.github/workflows/frontend-image.yml` |

## Run

```bash
cp .env.example .env    # the published images; change BACKEND_IMAGE and FRONTEND_IMAGE for others
docker compose pull
docker compose up -d
```

Open http://localhost:4200 and sign in as `customer@appointments.test`, `staff@appointments.test` or
`manager@appointments.test` (password `Local-dev-password-1`). The first start takes a few minutes: Keycloak imports
the realm and the backend loads the demo data.

| Page | Address |
|---|---|
| App | http://localhost:4200 |
| Swagger UI | http://localhost:8081/swagger-ui.html |
| Keycloak admin | http://localhost:8180/admin (admin / admin) |
| Mailpit (emails) | http://localhost:8025 |
| Grafana | http://localhost:3000 |

Stop with `docker compose down`; add `-v` to also delete the database, Keycloak and monitoring data.

To try images built on your machine instead, build them in each project (`docker build -t
branch-appointments-backend:local .`, likewise for the frontend) and point `.env` at those tags.

## How it fits together

- The browser talks only to the frontend (nginx), which serves the app and proxies `/api` to the backend.
- Keycloak is at `http://localhost:8180`, the address built into the frontend and in every token. The backend checks
  tokens against that address but calls Keycloak by its service name (`KEYCLOAK_URL`).
- The backend reads its secrets from Vault. `vault-init` seeds a dev-mode Vault on every start and writes the
  secrets Keycloak and postgres-exporter need into `.secrets/` (git-ignored).

## Notes

- Local only: dev-mode Vault with a fixed root token, test users, demo data, no HTTPS.
- It uses the same host ports as the backend project's `compose.yaml`, `mvn spring-boot:run` and `npm start`; stop
  those first.
- `keycloak/`, `vault/init.sh` and `monitoring/` are copies of the backend project's. Copy them again when they change
  there.
