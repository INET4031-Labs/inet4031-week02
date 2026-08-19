## Week 2: Building Your Three-Tier Application Stack

**Sprint 1 Async | Due before Sprint 1 Review**

### Overview

In this lab, your team containerizes and operates the incident tracking application: a Python Flask API connected to a PostgreSQL database and fronted by an Nginx reverse proxy. The application code is provided. Your job is to write the Docker Compose configuration that wires the three services together, handle startup ordering, configure data persistence, and manage credentials without hardcoding them. You will also extend the Ansible playbook to include the application stack, moving one step closer to the full automated rebuild your team will demonstrate at the end of the semester. After completing this lab, you will have a running three-tier application stack defined entirely in code, with credentials handled through environment variable injection and data persistence verified across container restarts.

### Learning Objectives

- Write a Docker Compose file that defines a three-tier application with health checks and explicit dependency ordering
- Configure a named network and named volume for service communication and data persistence
- Implement an environment variable pattern for credential management that never commits secrets to version control (GitHub)
- Verify that the application stack survives container restarts without data loss
- Extend the Ansible playbook with a role that brings up the Docker Compose stack as part of the rebuild process

### Prerequisites

- Week 1 complete: GitHub repo exists, team container is accessible to all members, `ansible/site.yml` is committed and runs clean
- Docker is running inside the team container (`docker info` returns output without error)
- Application source code provided by the professor is available at the path specified in lab materials

### Sprint 1 Context

This is async work. The Scrum Master owns the sprint board and keeps it updated. Pull tickets as you work. Coordinate so each role contributes to its section. **No single team member should complete the entire lab.**

- **System Admin:** leads Part 1 (service definition) and Part 3 Ansible addition
- **Developers:** write and test the Docker Compose file, configure networking and volumes in Part 2
- **QA:** runs all validation checks, is the final approver before deliverables are marked Done
- **Scrum Master:** keeps the board current, resolves blockers, participates in at least one Part

---

### Part 1: Define Your Services

The incident tracking application has three services that must run together. You are not writing the application code. You are writing the configuration that runs it.

**The three services:**

- **PostgreSQL** (`db`): the database. Stores incident records. Must finish initializing before Flask can connect.
- **Flask** (`flask`): the API. Reads and writes incidents. Must be healthy before Nginx routes traffic to it.
- **Nginx** (`nginx`): the reverse proxy. Handles incoming requests and forwards them to Flask. Comes up last.

**Step 1.** Inside your team container, navigate to the repo and create the directory for this week's files.

```bash
cd inet4031-team-[number]
mkdir -p week-2/app
```

**Step 2.** Place the provided application source code in `week-2/app/`. The professor will specify the exact path or repository URL. Confirm the files are present.

```bash
ls week-2/app/
```

Expected output includes at minimum: `Dockerfile`, the Flask application file ( `app.py`), and `requirements.txt`. If any of these are missing, check with the professor before continuing.

**Step 3.** Create `week-2/docker-compose.yml` and copy and paste the following Docker compose file into your repository. Begin with the database service only.

```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
```

**Step 4.** Add the Flask service to `docker-compose.yml` below the `db` definition.

```yaml
  flask:
    build: ./app
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
```

**Step 5.** Create `week-2/nginx.conf`. Nginx needs this file to know where to send traffic.

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://flask:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Step 6.** Add the Nginx service to `docker-compose.yml`.

```yaml
  nginx:
    image: nginx:alpine
    ports:
      - "${HOST_PORT:-8080}:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      flask:
        condition: service_healthy
    networks:
      - app-network
```

**Discussion (answer in Google Doc, Sprint 1 Week 2 section):**

- What order do these services need to start in? Can Nginx come up before Flask is ready? Can Flask connect to PostgreSQL before it finishes initializing? What does `condition: service_healthy` enforce that plain `depends_on` does not?
- What is the difference between a container being "running" and a container being "healthy"? Why does that distinction matter when one service depends on another?

---

### Part 2: Networking and Persistence

Without explicit network and volume configuration, services cannot reliably find each other and data disappears when containers stop.

**Step 1.** Add the top-level `networks` and `volumes` definitions to the bottom of `week-2/docker-compose.yml`.

```yaml
networks:
  app-network:
    driver: bridge

volumes:
  db-data:
```

**Step 2.** You need a `.env` file before you can start the stack. Copy the example file as a placeholder.

```bash
cp week-2/.env.example week-2/.env
```

Ensure your `.env` has the following contents. If not, then copy and paste it into your file:

```
POSTGRES_DB=statustracker
POSTGRES_USER=appuser
POSTGRES_PASSWORD=changeme
HOST_PORT=8080
```

**Step 3.** Start the stack for the first time.

```bash
cd week-2
docker compose up -d
```

Wait 30 seconds for health checks to run before checking status.

**Step 4.** Check the status of all containers.

```bash
docker compose ps
```

All three containers should show `healthy` in the Status column. If any show `starting` after 60 seconds, check the logs for that service.

**Step 5.** If any container is not healthy, check its logs to find the error.

```bash
docker compose logs db
docker compose logs flask
docker compose logs nginx
```

**Step 6.** Test that data persists across a container restart. First, create a test incident by calling the Flask API through Nginx.

```bash
curl -X POST http://localhost:8080/incidents \
  -H "Content-Type: application/json" \
  -d '{"title": "Persistence check", "status": "open", "description": "this should survive a restart"}'
```

Note the ID value in the response.

**Step 7.** Restart only the PostgreSQL container (not the whole stack).

```bash
docker compose restart db
```

Wait 20 seconds, then retrieve all incidents.

```bash
curl http://localhost:8080/incidents
```

Your test incident should appear in the response. If it does not, the named volume is not working correctly.

**Step 8.** Stop the stack and bring it back up without destroying the volume.

```bash
docker compose down
docker compose up -d
```

After the stack is healthy, retrieve incidents again. Your test incident should still be present.

**Step 9.** Observe what happens when you destroy the volume intentionally.

```bash
docker compose down -v
docker compose up -d
```

Retrieve incidents after the stack is healthy. The database is empty. This is expected. The `-v` flag removes named volumes along with containers.

> **Enterprise Pattern:** In production, `docker compose down -v` is a destructive operation that causes data loss. Operations teams protect against accidental data loss by running database backups before planned downtime and by separating the database lifecycle from the application lifecycle. You will work with backup tooling in Sprint 4.

**Discussion (answer in Google Doc):**

- What happens to your database data after `docker compose down`? What about `docker compose down -v`? When would each command be appropriate to use?
- Nginx refers to Flask as `flask:5000` in its configuration. How does Docker resolve the hostname `flask` to an IP address? What would happen if you renamed the Flask service to `api` but forgot to update `nginx.conf`?

---

### Part 3: Environment Configuration and Ansible

**Step 1.** Add `.env` to `.gitignore` so it is never committed. If `.gitignore` is not already present in your repository, then create the file in your repo root and add `.env` to it.

```bash
echo "week-2/.env" >> .gitignore
```

Verify the rule is working:

```bash
git check-ignore -v week-2/.env
```

Expected output: a line showing which `.gitignore` rule matches `week-2/.env`.

Commit the `.gitignore` update:

```bash
git add .gitignore week-2/.env.example
git commit -m "chore: ignore .env files, add .env.example for week-2"
git push origin main
```

> **Enterprise Pattern:** The `.env.example` pattern is a standard way to document required configuration without committing secrets. In production, these values would come from a secrets manager rather than a local file, but the pattern is the same: document what is needed, inject actual values at runtime, never store actual secrets in version control.

**Step 2.** Add the `app-stack` role to `ansible/site.yml`. This role will bring up the Docker Compose stack as part of the automated rebuild playbook that runs on Demo Day.

Create the role directory structure:

```bash
mkdir -p ansible/roles/app-stack/tasks
```

Create `ansible/roles/app-stack/tasks/main.yml`:

```yaml
---
- name: Ensure .env file exists for app stack
  copy:
    src: "{{ playbook_dir }}/../week-2/.env.example"
    dest: "{{ playbook_dir }}/../week-2/.env"
    force: no

- name: Bring up the Docker Compose application stack
  community.docker.docker_compose_v2:
    project_src: "{{ playbook_dir }}/../week-2"
    state: present
  become: yes
```

The `force: no` on the copy task means the playbook will create `.env` from the example if it does not exist, but will not overwrite an existing `.env`.

**Step 3.** Add the new role to `ansible/site.yml`. Open the file and append a second play below the existing baseline play.

```yaml
- name: Deploy application stack
  hosts: localhost
  connection: local
  become: yes

  roles:
    - app-stack
```

**Step 4.** Install the Ansible collection that provides the `docker_compose_v2` module.

```bash
ansible-galaxy collection install community.docker
```

**Step 5.** Run the full playbook to confirm both plays execute without error.

```bash
ansible-playbook -i ansible/inventory ansible/site.yml
```

Check the `PLAY RECAP` section. Both plays should show `failed=0` and `unreachable=0`.

**Step 6.** Commit all Ansible changes.

```bash
git add ansible/
git commit -m "feat: add app-stack role to site.yml for Docker Compose stack"
git push origin main
```

**Discussion (answer in Google Doc):**

- Where should credentials live in a containerized application? What are the risks of hardcoding them directly in `docker-compose.yml`?
- If a teammate cloned your repo and ran `docker compose up`, what would they need to provide that is not in the repo? What file should they consult to know exactly what is required?

---

### Storage Check

Run these commands inside the team container and record the output in your Google Doc under "Week 2 Storage Check."

```bash
df -h
docker system df
```

Compare `docker system df` output to your Week 1 baseline. You should now see images, at least three containers, and one named volume listed.

---

### Validation Checks

#### Validation Check: All Three Services Are Healthy

Run from inside the `week-2/` directory:

```bash
docker compose ps
```

Expected output shows three rows, each with `healthy` in the Status column. If a service shows `starting` after 60 seconds: run `docker compose logs [service-name]`.

#### Validation Check: Nginx Is Reachable on the Mapped Port

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/
```

Expected output: `200`. If you see `000` or `Connection refused`: Nginx is not running or the port mapping is wrong.

#### Validation Check: Data Persists Across Container Restart

Create a test incident, restart the database container only, retrieve all incidents, and confirm your record is present.

#### Validation Check: Ansible Playbook Runs Clean

```bash
ansible-playbook -i ansible/inventory ansible/site.yml
```

Expected: `PLAY RECAP` shows `failed=0` and `unreachable=0` for both plays.

#### Validation Check: Check Script Passes

```bash
./scripts/check-week2.sh
```

---

### Deliverables

- `week-2/docker-compose.yml` committed (all three services, health checks, named network, named volume)
- `week-2/.env.example` committed
- `week-2/nginx.conf` committed
- `.gitignore` updated so `week-2/.env` is excluded
- `week-2/README.md` committed (explains the stack, how to bring it up, what `.env` values are required)
- `ansible/site.yml` updated with the `app-stack` role play
- `ansible/roles/app-stack/tasks/main.yml` committed
- All validation checks pass
- `./scripts/check-week2.sh` runs clean inside the container
- Google Doc updated with Sprint 1 Week 2 reflection answers and Week 2 storage check values

**Screenshot requirements:**

- **Screenshot 1:** `docker compose ps` showing all three services with `healthy` status
- **Screenshot 2:** `curl` response from the `/health` endpoint
- **Screenshot 3:** Persistence check: incident created, database container restarted, incident retrieved successfully
- **Screenshot 4:** `ansible-playbook` PLAY RECAP showing `failed=0` for both plays
- **Screenshot 5:** `./scripts/check-week2.sh` passing

---

### Reflection Questions (Answer in Google Doc)

1. What did you learn about container startup ordering that you did not expect? If you removed `condition: service_healthy` and left only `depends_on: db`, what would change about when Flask starts?
2. If your team's container were wiped today, which parts of this stack could be rebuilt automatically from your repo? Which parts would require manual steps, and why?
3. Docker Compose manages containers on one machine. What would break if you needed this application to run across multiple machines? What problem does that create for the PostgreSQL container in particular?
4. Your `.env` file holds database credentials. Trace the path of those credentials: where are they stored on disk, how do they reach the running container process, and at what points could they be exposed?

---

---

