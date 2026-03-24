# AMΔDEUS

**Private homelab infrastructure — self-hosted, zero cloud dependency.**

A Proxmox-based infrastructure running a K3s Kubernetes cluster with dual monitoring, centralized logging, DNS-level threat blocking, and self-hosted applications. Everything runs on-premise with end-to-end encryption and remote access via Tailscale VPN.

> **[Live Presentation](https://kevi-cyber-beep.github.io/amadeus-infra/)** — interactive overview of the full stack.

---

## Architecture

```
Internet ──→ WARP Tunnel ──→ AdGuard DNS ──→ Proxmox Hypervisor
                                                  │
                              ┌────────────────────┼────────────────────┐
                              │                    │                    │
                         K3s Cluster          Monitoring VM        LXC Containers
                         (5 namespaces)       (Podman stack)      (VPN, DNS, Tools)
                              │                    │
                    ┌─────────┼─────────┐    Grafana + Prometheus
                    │         │         │    + Loki (7d retention)
                 Miniflux  SearXNG  Calibre-Web
                 ArgoCD    Grafana  Prometheus
```

**Hosts:** 5 virtual servers (VMs + LXCs) on a single Proxmox node, isolated by function.

## Stack

| Layer | Technology |
|-------|-----------|
| **Hypervisor** | Proxmox VE (QEMU/KVM + LXC) |
| **Orchestration** | K3s (lightweight Kubernetes) |
| **GitOps** | ArgoCD — continuous delivery from Git |
| **Ingress + TLS** | Traefik + cert-manager (self-signed CA, valid 10 years) |
| **DNS Filtering** | AdGuard Home — 329K+ blocked domains across 12 blocklists |
| **Monitoring** | Dual Grafana + Prometheus (infra-level and K8s-level, separate stacks) |
| **Logging** | Loki + Promtail DaemonSet (7-day retention, all namespaces) |
| **VPN** | Tailscale subnet router (zero-trust remote access) |
| **Containers** | Rootless Podman (monitoring VM) + K3s pods |

## Services

| Service | Description | Platform |
|---------|-------------|----------|
| **Miniflux** | Self-hosted RSS reader (27 feeds, PostgreSQL StatefulSet) | K3s |
| **SearXNG** | Private metasearch engine, no tracking | K3s |
| **Calibre-Web** | E-book library with OPDS catalog and web reader | K3s |
| **ArgoCD** | GitOps CD — every commit triggers deployment | K3s |
| **Grafana (x2)** | Infrastructure dashboards + Kubernetes dashboards | K3s + Podman |
| **Prometheus (x2)** | Metrics collection with 7 alert rules | K3s + Podman |
| **Loki** | Centralized log aggregation from all K8s namespaces | Podman |
| **AdGuard Home** | DNS-level ad/malware/phishing blocker | LXC |
| **Metadata Inspector** | ExifTool + FastAPI — file metadata analysis with auto-delete | LXC |
| **Tailscale** | VPN gateway with subnet routing to entire lab | LXC |

## Security Layers

- **Network:** Tailscale VPN — zero public ports exposed
- **DNS:** 329,392 rules blocking ads, malware, phishing, crypto miners, stalkerware
- **TLS:** Self-signed Root CA (RSA 4096, valid 2026–2036) on all services via cert-manager
- **Isolation:** LXC unprivileged containers + K3s namespace separation
- **Rate limiting:** Nginx (5 req/min) on metadata service
- **File security:** Auto-delete after analysis, UUID filenames, no persistence
- **Monitoring:** 7 alert rules (PodCrashLooping, HighCPU, HighMemory, DiskSpaceLow, NodeDown, etc.)
- **Firewall:** UFW deny-all inbound on all hosts

## Monitoring Architecture

**Dual-stack design** — infrastructure and Kubernetes monitored separately:

| Stack | Location | Monitors | Dashboards |
|-------|----------|----------|------------|
| **Grafana #1** | Dedicated VM (Podman) | Host-level CPU, RAM, disk, network for all nodes | Node Exporter Full, AdGuard DNS |
| **Grafana #2** | Inside K3s cluster | Pod lifecycle, container resources, K8s API metrics | kube-prometheus-stack defaults |
| **Loki** | Dedicated VM (Podman) | All K8s pod logs via Promtail DaemonSet | LogQL queries in both Grafana instances |

## Documentation

This repo contains the live presentation website. Full operational documentation (setup guides, runbooks, troubleshooting) is maintained separately and covers:

- 11 step-by-step deployment guides
- 4 incident response runbooks (cluster-down, DNS failure, monitoring failure, networking)
- 4 troubleshooting guides (Kubernetes, ArgoCD, networking, services)
- Operations manual with 70+ commands
- Per-service configuration reference

## Tech

Built and maintained by **Keviano Gjonaj** — March 2026.

Website uses a custom dark UI with matrix rain effects, terminal boot sequence, and interactive SVG network diagrams. Single HTML file, no build tools, no frameworks.
