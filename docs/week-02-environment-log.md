# Environment Log

**Owned by:** System Admin

This log records the state of the team's container environment at the start and end of each sprint. Use this to track disk usage, resource allocation, snapshot status, and any environment configuration decisions.

This file is appended to (not rewritten) each sprint. Include a dated entry for each sprint checkpoint.

---

## Sprint 1: Weeks 1-2

### Week 1 Baseline (from sprint kickoff)

**Date:** TODO: Record the date

**OS and Disk:**
```
TODO: Paste output of: cat /etc/os-release
TODO: Paste output of: df -h
```

**Installed Tools:**
```
TODO: Paste output of: which docker git python3 curl ansible
```

**Docker Daemon Status:**
```
TODO: Paste output of: docker info
```

**Notes:** Any issues or observations about the baseline environment.

### Week 2 End-of-Sprint (from async completion)

**Date:** TODO: Record the date

**Disk and Resource State:**
```
TODO: Paste output of: df -h
TODO: Paste output of: docker system df
```

**Docker Compose Stack Status:**
```
TODO: Paste output of: docker compose -f week-2/docker-compose.yml ps
```

**Environment Decisions Made This Sprint:**

TODO: Record any decisions about environment configuration, resource allocation, or container management.

**Issues Encountered:**

TODO: Log any environment-related problems encountered and how they were resolved.

**Snapshot Confirmation:**

TODO: Confirm whether a VM/container snapshot was taken and note the timestamp.

---

## Sprint 2: Weeks 3-4

TODO: Add entries for this sprint when you reach Week 3.

---

*Template note: Continue appending entries for each subsequent sprint. Do not delete or reorganize previous sprint data.*
