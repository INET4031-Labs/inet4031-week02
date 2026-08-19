# Week 2: Building Your Three-Tier Application Stack

**Sprint 1 Async | Due before Sprint 1 Review**

[Full Week 2 content - lines 423-843 from Lab Directions document follow in next message due to size constraints]

Please refer to the [INET 4031 Lab Directions - Full Curriculum (Proposed)](../../Documents/INET%204031%20Lab%20Directions%20-%20Full%20Curriculum%20(Proposed).md) document for the complete Week 2 lab directions, or this summary:

## Overview

In this lab, your team containerizes and operates the incident tracking application: a Python Flask API connected to a PostgreSQL database and fronted by an Nginx reverse proxy. The application code is provided. Your job is to write the Docker Compose configuration that wires the three services together, handle startup ordering, configure data persistence, and manage credentials without hardcoding them. You will also extend the Ansible playbook to include the application stack, moving one step closer to the full automated rebuild your team will demonstrate at the end of the semester. After completing this lab, you will have a running three-tier application stack defined entirely in code, with credentials handled through environment variable injection and data persistence verified across container restarts.

## Key Parts
- **Part 1**: Define Your Services (PostgreSQL, Flask, Nginx)
- **Part 2**: Networking and Persistence
- **Part 3**: Environment Configuration and Ansible
- **Storage Check**: Compare docker system df to Week 1 baseline
- **Validation Checks**: Service health, persistence, Ansible playbook
- **Deliverables**: docker-compose.yml, .env.example, nginx.conf, app-stack role

See the full Lab Directions document for complete step-by-step instructions, code examples, validation checks, and reflection questions.
