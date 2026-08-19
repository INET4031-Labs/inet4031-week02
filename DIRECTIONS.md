# Week 2: Building Your Three-Tier Application Stack

**Sprint 1 Async | Due before Sprint 1 Review**

## Overview
Containerize the incident tracking application using Docker Compose: Nginx reverse proxy, Flask API, PostgreSQL database.

## Key Activities
- Docker Compose service definitions with health checks and dependency ordering
- Named networks and volumes for persistence and communication
- Environment variable pattern for credential management (.env files)
- Ansible app-stack role for automated deployment

## Learning Objectives
- Write Docker Compose configuration for three-tier stack
- Configure named networks and volumes for persistence
- Manage credentials through environment variables without version control
- Extend Ansible playbook with application deployment layer

## Deliverables
- docker-compose.yml with all three services, health checks, named network/volume
- .env.example and .gitignore configuration
- nginx.conf for reverse proxy configuration
- app-stack Ansible role
- Data persistence verified across container restarts

## Key Sections
- **Part 1**: Define services (PostgreSQL, Flask, Nginx) with health checks
- **Part 2**: Configure networking and persistence
- **Part 3**: Environment configuration and Ansible integration
- **Storage Check**: docker system df output showing images and volumes
- **Validation Checks**: All services healthy, Nginx reachable, data persists, Ansible playbook passes

**Status**: Application layer - Three-tier stack running in containers

---
*For complete step-by-step instructions, refer to the INET 4031 Lab Directions - Full Curriculum document in the course materials.*
