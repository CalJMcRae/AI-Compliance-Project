# Eramba Community: local setup notes

Eramba Community Edition, self-hosted via Docker, is the GRC platform this assessment is run
in. It is free (no licence fee, no user limit, no time limit). These notes record how the
instance was stood up; the Docker files themselves live outside this repo and are not
committed.

## Requirements

- Docker Desktop running (WSL2 backend on Windows)
- ~8 GB RAM available to Docker, ~3 GB disk
- Git configured to keep LF line endings for the clone (see below)

## Install

Run in a terminal, **not** inside this repo:

```bash
cd ~
# keep LF endings so the container entrypoint scripts work on Windows
git clone --config core.autocrlf=false https://github.com/eramba/docker eramba-docker
cd eramba-docker
```

Edit `.env`:

```
DB_PASSWORD='<a strong db password>'
MYSQL_ROOT_PASSWORD='<a different strong password>'
PUBLIC_ADDRESS=https://localhost:8443
```

(Wrap passwords in single quotes if they contain `$` or other shell characters.)

Start it:

```bash
docker compose -f docker-compose.simple-install.yml up -d
docker logs -f eramba
```

If the compose filename is not found, run `ls docker-compose*.yml` and use the
simple-install one that is present.

Wait for the logs to show Apache started and initialisation complete, then open
**https://localhost:8443** (accept the self-signed certificate warning).

## First run

1. Choose "Unlock power of GRC", set an admin password, admin email, and country.
2. Community edition asks for a free activation token. Request it from eramba.org if it is
   not emailed automatically, then paste it in.
3. Settings > System & Maintenance > System Health: everything should read OK. If the cron
   or queue items are red, note it here; the assessment does not depend on scheduled jobs.
4. Settings > Application Configuration > Localization: set the time zone.

## What gets built here (mapping to the repo)

| Eramba module | Repo source |
|---|---|
| Controls catalogue | [`../03-control-catalogue/control-catalogue.csv`](../03-control-catalogue/control-catalogue.csv) |
| Risk register | [`../04-ai-risk-register/ai-risk-register.csv`](../04-ai-risk-register/ai-risk-register.csv) |
| Control assessment / audits | [`../05-assessment-and-gaps/`](../05-assessment-and-gaps/) |
| Treatment / risk mitigation | [`../06-treatment-plan/`](../06-treatment-plan/) |

Curated exports and screenshots go in [`eramba-exports/`](eramba-exports/) and
[`screenshots/`](screenshots/).

## Teardown

```bash
cd ~/eramba-docker
docker compose -f docker-compose.simple-install.yml down -v   # -v also drops the data volume
```

## Setup log

`TODO` record: date stood up, Eramba version, any System Health warnings, activation token
obtained (y/n).
