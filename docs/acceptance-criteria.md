# Acceptance Criteria for Sprint 1 Week 2

**Owned by:** QA

This document defines what "done" means for this sprint, written before developers implement. It goes beyond what the check script validates and captures the team's understanding of what a working solution looks like.

This file is completed during the synchronous week (Week 1) in preparation for the async week (Week 2) implementation.

---

## Part 1: Service Definition

**Acceptance Criteria:**

TODO: QA writes specific, testable criteria for the Docker Compose services:
- Are all three services (db, flask, nginx) defined?
- Do health checks work as intended?
- Are dependencies properly ordered with `condition: service_healthy`?
- Can the System Admin run the commands from the lab without modification?

**Example format:**
- [ ] PostgreSQL service starts and becomes healthy without manual intervention
- [ ] Flask service waits for PostgreSQL health before starting
- [ ] Flask health check correctly validates the API endpoint
- [ ] Nginx service waits for Flask health before starting

## Part 2: Networking and Persistence

**Acceptance Criteria:**

TODO: QA writes specific criteria for networking and data persistence:
- Does the named network exist and connect all three services?
- Can services communicate by hostname (e.g., `flask:5000`)?
- Does the named volume persist data across container restarts?
- Does `docker compose down -v` correctly destroy the volume?

**Example format:**
- [ ] All three containers can resolve each other's hostnames
- [ ] Test data persists after `docker compose restart db`
- [ ] Test data is lost after `docker compose down -v`
- [ ] Multiple team members can run the stack simultaneously without port conflicts

## Part 3: Environment Configuration and Ansible

**Acceptance Criteria:**

TODO: QA writes criteria for .env handling and Ansible integration:
- Is `.env` excluded from version control?
- Does `.env.example` document all required variables?
- Does the Ansible playbook bring up the Docker Compose stack without error?
- Does the playbook run idempotently (no changes on second run)?

**Example format:**
- [ ] `git check-ignore -v week-2/.env` shows the file is ignored
- [ ] `.env.example` contains POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD, HOST_PORT
- [ ] `ansible-playbook` runs both plays (baseline and app-stack) with `failed=0`
- [ ] Running the playbook twice produces `changed=0` on the second run

## Validation and Delivery

**Acceptance Criteria:**

TODO: QA writes criteria for what validation success looks like:
- Do all validation checks pass without manual workarounds?
- Does the check script run clean?
- Are all required files committed to the repository?
- Has the team documented findings in the Google Doc?

**Example format:**
- [ ] `docker compose ps` shows three services all with "healthy" status
- [ ] `curl http://localhost:8080/` returns HTTP 200
- [ ] Persistence test incident appears after database restart
- [ ] `./scripts/check-week2.sh` passes with no errors
- [ ] Google Doc contains Sprint 1 Week 2 reflections and storage check

---

## Known Gaps or Ambiguities

TODO: If there are aspects of the lab that QA finds unclear or potentially problematic, document them here for discussion with the team:

- Is there anything in the lab directions that is ambiguous?
- Are there edge cases not covered by the check script?
- Are there dependencies on external resources (e.g., professor-provided code) that could block the team?

---

## Sign-Off

TODO: QA approves the sprint work against these criteria:

**QA Name:** ______________________  
**Date Signed:** ______________________  
**Notes:** Any final observations before the sprint begins.
