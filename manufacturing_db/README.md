manufacturing_db (PostgreSQL) - Local/Dev Guide

Overview
- Core PostgreSQL database for the Manufacturing Cloud Suite.
- The backend API connects using either POSTGRES_URL or POSTGRES_USER/POSTGRES_PASSWORD/POSTGRES_DB/POSTGRES_HOST/POSTGRES_PORT.

Required tools (choose one path)
- Docker/Docker Desktop (recommended) OR
- Local PostgreSQL 14+ (15+ recommended)
- psql client for connectivity tests

Environment setup
- Copy .env.example to .env and adjust values:
  - POSTGRES_USER
  - POSTGRES_PASSWORD
  - POSTGRES_DB
  - POSTGRES_HOST
  - POSTGRES_PORT (5432 typical; included startup.sh uses 5000)
  - POSTGRES_URL (optional, overrides individual vars)

Startup options

A) Docker (recommended)
- Quick run:
  docker run --name mfg-db -e POSTGRES_USER=appuser -e POSTGRES_PASSWORD=dbuser123 -e POSTGRES_DB=myapp -p 5432:5432 -d postgres:15
- Use the same values in API .env.

B) Docker Compose (sample)
version: "3.9"
services:
  db:
    image: postgres:15
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-appuser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-dbuser123}
      POSTGRES_DB: ${POSTGRES_DB:-myapp}
    volumes:
      - db_data:/var/lib/postgresql/data
volumes:
  db_data:

C) Local PostgreSQL install (advanced)
- Ensure the server is running and create user/database matching .env.
- To test connectivity:
  psql "postgresql://appuser:dbuser123@localhost:5432/myapp"

Note on startup.sh (included script)
- manufacturing_db/startup.sh attempts to bootstrap a local Postgres using system paths and sudo; this may not work in all environments. Prefer Docker or a standard local Postgres install.
- If you do use startup.sh, note it defaults to port 5000. Adjust API .env accordingly.

Connection info for API
- API expects either:
  - POSTGRES_URL=postgresql://USER:PASSWORD@HOST:PORT/DB
  - or individual POSTGRES_* variables

Troubleshooting
- Port already in use:
  - Another Postgres may be running. Change the host port mapping (e.g., 5433:5432) and update API .env.
- Authentication failed:
  - Verify POSTGRES_USER/POSTGRES_PASSWORD and that the user has privileges on POSTGRES_DB.
- Connection refused:
  - Verify container is running and port is exposed, firewall allows connections, and HOST/PORT are correct.
- SSL or driver errors from API:
  - Ensure API uses asyncpg via the URL scheme postgresql+asyncpg when necessary (API handles this automatically from POSTGRES_URL or individual vars).
