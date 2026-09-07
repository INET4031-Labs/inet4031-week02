# Week 2: Building Your Three-Tier Application Stack

**Sprint 1 | Due before Sprint 1 Review**

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
repo, you write it yourself as part of the lab.

## Role Distribution

- **Scrum Master:** keeps the sprint board current, resolves blockers, participates in at least one Part
- **System Admin:** leads service definition and the Ansible addition
- **QA:** runs all validation checks, is the final approver before deliverables are marked Done
- **Developers:** write and test the Docker Compose file, configure networking and volumes

## Deliverables

- `docker-compose.yml` (all three services, health checks, named network, named volume)
- `.env.example` (template with example credentials)
- `nginx.conf` (reverse proxy configuration)
- `ansible/site.yml` updated with the `app-stack` role play
- `ansible/roles/app-stack/tasks/main.yml` committed
- All validation checks pass
- `./scripts/check-week2.sh` runs clean inside the container
- Google Doc updated with Sprint 1 Week 2 reflection answers and Week 2 storage check values

## Full Instructions

The complete step-by-step lab, including exact Docker Compose configuration, validation commands, and discussion questions, is in this repo's Wiki tab. This README is a reference, not a substitute.
