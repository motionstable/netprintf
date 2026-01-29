# Dedicated Multi-Server Web Hosting Setup  
## Extended User Manual (Low-Maintenance, Long-Life Configuration)

This document describes the **complete operational setup** for a self-hosted web infrastructure consisting of **8 dedicated backend servers + 1 reverse proxy**, all publicly accessible via a **single static IP**, without virtualization, VPS, or cloud dependencies.

The goal is **stability, predictability, and minimal maintenance over years**.

---

## Table of Contents

1. Purpose and Scope  
2. Architecture Overview  
3. Network Design  
4. Power and Outage Safety Model  
5. Operating System Baseline  
6. Reverse Proxy Configuration  
7. Backend Server Configuration  
8. DNS and Public Access  
9. Security Model  
10. Updates and Maintenance Strategy  
11. Backup and Recovery Strategy  
12. Monitoring and Alerting  
13. Failure Scenarios and Recovery  
14. Troubleshooting Guide  

---

## 1. Purpose and Scope

This setup is designed for:
- Long-running public web services
- Predictable performance
- Easy replacement of failed components
- Minimal ongoing administrative effort

This manual **intentionally avoids hardware specifications** and focuses on **configuration, behavior, and operational rules**.

---

## 2. Architecture Overview

### Logical roles
- **Edge Router / Firewall**  
  Handles WAN connection, NAT, and port forwarding.
- **Reverse Proxy Server**  
  Terminates TLS, routes requests to backend servers.
- **8 Backend Servers**  
  Each runs exactly one primary service or site.

### Design principles
- One role per machine
- No shared state between backends
- Single public ingress point
- Static addressing everywhere

---

## 3. Network Design

### IP layout
- Use a single private subnet
- Assign **static IPs** to all infrastructure
- Do not use dynamic addressing for servers

### Traffic flow
```

Internet
↓
Router (Static Public IP)
↓
Reverse Proxy
↓
Backend Servers (internal only)

```

### Port exposure rules
- Only ports 80 and 443 are exposed publicly
- Backend servers never accept direct WAN traffic

---

## 4. Power and Outage Safety Model

### Objectives
- Prevent data corruption
- Ensure automatic recovery after outages
- Avoid manual intervention

### Key behaviors
- All systems must:
  - Shut down cleanly during extended outages
  - Automatically power on when electricity returns

### Operational rules
- Network infrastructure must stay online longer than backend servers
- Shutdown order:
  1. Backend servers
  2. Reverse proxy
  3. Network devices (last)

This ensures graceful service degradation instead of sudden failure.

---

## 5. Operating System Baseline

### OS selection
- Use a long-term support Linux distribution
- Minimal installation (no desktop environment)

### Baseline configuration (all servers)
- Static IP configuration
- Time synchronization enabled
- SSH key authentication only
- Root login disabled
- Automatic **security-only** updates enabled

### Consistency rule
All backend servers must:
- Use the same OS version
- Have identical baseline configuration
- Differ only by hosted application

---

## 6. Reverse Proxy Configuration

### Responsibilities
- TLS termination
- HTTP → HTTPS redirection
- Domain-to-backend routing
- Rate limiting
- Centralized logging

### Routing model
- Each domain maps to exactly one backend server
- No load balancing between backends (by design)

### TLS management
- Use automated certificate issuance and renewal
- Certificates are stored only on the reverse proxy

### Reload strategy
- Configuration changes must support zero-downtime reloads
- Never restart the proxy during peak traffic

---

## 7. Backend Server Configuration

### Service isolation
- One primary service per server
- No shared databases across servers unless explicitly required

### Network access
- Backend servers:
  - Accept traffic only from the reverse proxy
  - Do not expose SSH or application ports publicly

### Application deployment
- Prefer simple, reproducible deployment methods
- Avoid auto-upgrading frameworks
- Pin application versions deliberately

---

## 8. DNS and Public Access

### DNS model
- All domains resolve to the same public IP
- Routing is handled exclusively by the reverse proxy

### Benefits
- No DNS changes required when replacing backend servers
- Easy migration between backends
- Simple rollback capability

---

## 9. Security Model

### Network security
- Default-deny firewall policy
- Explicit allow rules only
- No direct backend exposure

### Access control
- SSH access restricted to trusted internal networks
- No password authentication

### Principle
Security is achieved through **simplicity and isolation**, not complexity.

---

## 10. Updates and Maintenance Strategy

### Automated
- OS security patches
- TLS certificate renewal
- Log rotation

### Manual (scheduled)
- OS major upgrades
- Application upgrades
- Firmware updates

### Maintenance cadence
- Quarterly reboots
- Annual configuration review
- Multi-year OS lifecycle planning

---

## 11. Backup and Recovery Strategy

### Backup scope
- Application data
- Databases
- Configuration files

### Exclusions
- OS installations
- Temporary files

### Backup rules
- Backups must be encrypted
- Backups must exist off-site
- Restore procedures must be tested

### Recovery philosophy
Servers are **replaceable**, data is not.

---

## 12. Monitoring and Alerting

### Minimal monitoring
- Host availability
- Disk usage thresholds
- Power state notifications

### Alert rules
- Alerts must be actionable
- Avoid excessive metrics
- Prefer reliability over detail

---

## 13. Failure Scenarios and Recovery

### Backend server failure
- Replace server
- Reinstall OS
- Restore application
- No DNS or proxy changes required

### Reverse proxy failure
- Restore configuration
- Reissue certificates if required
- Resume service

### Power outage
- Automatic shutdown
- Automatic restart
- No manual recovery expected

---

## 14. Troubleshooting Guide

### Issue: Site unreachable, proxy online
**Check**
- Backend server status
- Internal firewall rules
- Proxy routing configuration

### Issue: TLS certificate errors
**Check**
- System time synchronization
- Certificate renewal logs
- Domain name mismatch

### Issue: Backend reachable internally but not externally
**Check**
- Proxy forwarding rules
- Backend firewall restrictions
- Reverse proxy logs

### Issue: Servers do not restart after outage
**Check**
- BIOS power-restore settings
- Shutdown daemon configuration
- Power delivery order

### Issue: High load on reverse proxy
**Check**
- Misrouted traffic
- Unexpected domains
- Rate-limit configuration

---

## Closing Notes

This setup prioritizes:
- Predictable behavior
- Replaceable components
- Minimal administrative burden

If followed strictly, this architecture can run **for years** with only scheduled maintenance and no emergency interventions.

---
motionstable.
