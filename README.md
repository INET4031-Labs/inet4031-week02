# Week 2: Building Your Three-Tier Application Stack

## Overview

This week, your team containerizes and operates the incident tracking application: a Python Flask API connected to a PostgreSQL database and fronted by an Nginx reverse proxy. The application code is provided. Your job is to write the Docker Compose configuration that wires the three services together, handle startup ordering, configure data persistence, and manage credentials without hardcoding them.

## Prerequisites

- Week 1 complete: GitHub repo exists, team container is accessible to all members, `ansible/site.yml` is committed and runs clean
- Docker is running inside the team container (`docker info` returns output without error)
- Application source code is provided in this repo's `app/` directory (see "Pulling This Week's Starter Content" below)

## Pulling This Week's Starter Content Into Your Team Repo

This repo (`inet4031-week02`) is instructor-provided starter/reference content for
Week 2, not something you clone standalone. Pull the pieces you need into your
team's single repo (see Week 1's README for the one-repo-per-team model):

```bash
git remote add week2 https://github.com/INET4031-Labs/inet4031-week02.git
git fetch week2
git checkout week2/main -- .env.example app scripts docs
mkdir -p week-2
mv .env.example app week-2/
git remote remove week2
```

Do this before you start editing `week-2/` locally, or your local changes will be
silently overwritten by the checkout. `docs/` and `scripts/` land directly at your
repo root, same as every week. `week-2/nginx.conf` is not shipped as a file in this
repo -- you write it yourself as part of Step 2 (Configure Docker Compose) below.

## The Three Services

- **PostgreSQL** (`db`): the database. Stores incident records. Must finish initializing before Flask can connect.
- **Flask** (`flask`): the API. Reads and writes incidents. Must be healthy before Nginx routes traffic to it.
- **Nginx** (`nginx`): the reverse proxy. Handles incoming requests and forwards them to Flask. Comes up last.

## Getting Started

### Step 1: Place Application Source Code

The application source code is provided in the `app/` directory of this folder (pulled in the step above).

```bash
ls week-2/app/
```

Expected output includes at minimum: `Dockerfile`, the Flask application file (`app.py`), and `requirements.txt`.

### Step 2: Configure Docker Compose

Edit `docker-compose.yml` to define the three services (database, Flask, Nginx) with:
- Health checks for startup ordering
- Named network for service communication
- Named volume for database persistence
- Environment variable injection from `.env`

Refer to the lab directions for the exact configuration for each service.

### Step 3: Set Up Environment Variables

Copy `.env.example` to `.env`:

```bash
cp week-2/.env.example week-2/.env
```

This file holds database credentials and the port configuration. It must never be committed to version control.

### Step 4: Bring Up the Stack

```bash
cd week-2
docker compose up -d
```

### Step 5: Verify Health and Persistence

- All three containers should show `healthy` status
- Data should persist across container restarts (tested by the validation checks)

### Step 6: Extend Ansible

Add the `app-stack` role to `ansible/site.yml` so the Docker Compose stack can be brought up as part of the automated rebuild.

## Deliverables

- `docker-compose.yml` (all three services, health checks, named network, named volume)
- `.env.example` (template with example credentials)
- `nginx.conf` (reverse proxy configuration)
- `README.md` (this file, explains the stack and how to bring it up)
- `ansible/site.yml` updated with the `app-stack` role play
- `ansible/roles/app-stack/tasks/main.yml` committed
- All validation checks pass
- `./scripts/check-week2.sh` runs clean inside the container
- Google Doc updated with Sprint 1 Week 2 reflection answers and Week 2 storage check values

## Role Responsibilities

- **System Admin:** leads Part 1 (service definition) and Part 3 (Ansible addition)
- **Developers:** write and test the Docker Compose file, configure networking and volumes in Part 2
- **QA:** runs all validation checks, is the final approver before deliverables are marked Done
- **Scrum Master:** keeps the sprint board current, resolves blockers, participates in at least one Part

## Validation Checks

Run these after your stack is up:

```bash
# All three services healthy
docker compose ps

# Nginx reachable on mapped port
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/

# Data persists across restart
curl -X POST http://localhost:8080/incidents \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "status": "open", "description": "Persistence test"}'

# Ansible playbook runs clean
ansible-playbook -i ansible/inventory ansible/site.yml

# Check script passes
./scripts/check-week2.sh
```

## Discussion Questions

Answer these in your team Google Doc under "Sprint 1 Week 2 Reflections":

1. What order do these services need to start in? Can Nginx come up before Flask is ready? Can Flask connect to PostgreSQL before it finishes initializing? What does `condition: service_healthy` enforce that plain `depends_on` does not?

2. What is the difference between a container being "running" and a container being "healthy"? Why does that distinction matter when one service depends on another?

3. What happens to your database data after `docker compose down`? What about `docker compose down -v`? When would each command be appropriate to use?

4. Nginx refers to Flask as `flask:5000` in its configuration. How does Docker resolve the hostname `flask` to an IP address? What would happen if you renamed the Flask service to `api` but forgot to update `nginx.conf`?

5. Where should credentials live in a containerized application? What are the risks of hardcoding them directly in `docker-compose.yml`?

6. If a teammate cloned your repo and ran `docker compose up`, what would they need to provide that is not in the repo? What file should they consult to know exactly what is required?

## Storage Check

Run these commands and record output in your Google Doc under "Week 2 Storage Check":

```bash
df -h
docker system df
```

Compare `docker system df` output to your Week 1 baseline. You should now see images, at least three containers, and one named volume listed.
