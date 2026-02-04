---
sidebar_position: 11
title: Docker Monitoring
description: Monitor Docker Engine health, aggregate container states, and system resources.
---

# Docker Monitoring

This guide explains how to monitor your Docker Engine using **Harbor Lighthouse**. The agent connects directly to the local Docker daemon to stream real-time host-level statistics, including container state counts, system resource limits, and Swarm status.

**_Repo Link:_** https://github.com/harborscale/harbor-lighthouse

## Prerequisites

Before starting, ensure you have:

-   **Docker Engine** installed and running on the host.
-   **Harbor Lighthouse** installed (See [Installation Guide](/docs/lighthouse/)).
-   **Root or Docker Group** permissions for the user running the agent.

## How it Works

Unlike other solutions that deploy a "sidecar" container, Lighthouse runs as a lightweight binary on the host system. It communicates with the Docker Engine via the Unix socket (`/var/run/docker.sock` on Linux) or named pipe (on Windows). 

**Note on Performance:** The collector uses connection pooling to reuse the Docker client connection and explicitly disables heavy per-container disk usage calculations, ensuring minimal impact on your system even with hundreds of containers.

---

## Setup Guide

### 1. Configure Permissions (Linux/macOS)

To allow Lighthouse to read Docker stats, the user running the agent (usually your current user or root) must have access to the Docker socket.

If you are running Lighthouse as a specific user, add them to the `docker` group:

```bash
sudo usermod -aG docker $USER
```

> **Note:** You may need to log out and log back in (or restart the service) for these permissions to take effect.

**Windows:** Run PowerShell as Administrator when installing and running Lighthouse.

### 2. Add the Monitor

Use the `lighthouse` CLI to enable the Docker collector.

**For Harbor Scale Cloud (Linux/macOS):**

```bash
sudo lighthouse --add --name "docker-host-01" --harbor-id "YOUR_HARBOR_ID" --key "YOUR_API_KEY" --source docker
```

**For Harbor Scale Cloud (Windows):**

```powershell
lighthouse --add --name "docker-host-01" --harbor-id "YOUR_HARBOR_ID" --key "YOUR_API_KEY" --source docker
```

---

## Configuration

You can customize the Docker monitor using standard Lighthouse flags.

| Flag | Description | Default |
| --- | --- | --- |
| `--interval` | How often to poll engine stats (in seconds). | `60` |
| `--name` | The unique ID/Name for this Docker Host. | (Required) |
| `--source` | Must be set to `docker`. | - |

---

## Available Metrics

The `docker` source collects **Engine-level** metrics. It aggregates the states of all containers on the host rather than streaming stats for individual containers.

### Container Aggregates

| Metric | Description |
| --- | --- |
| **Total Containers** | Total number of containers on the host. |
| **Running** | Number of containers in `running` state. |
| **Exited/Stopped** | Number of containers in `exited` state. |
| **Paused** | Number of `paused` containers. |
| **Error/Dead** | Number of containers in `dead` or `restarting` states. |

### Engine & System Resources

| Metric | Description |
| --- | --- |
| **Images** | Total number of images stored locally. |
| **Volumes** | Total number of local Docker volumes. |
| **Goroutines** | Number of internal Docker goroutines (useful for debugging daemon load). |
| **CPUs** | Number of CPUs available to the Docker Engine. |
| **Total Memory** | Total memory available to the Docker Engine. |
| **File Descriptors** | Number of open file descriptors used by the daemon. |

### Swarm Mode

If the engine is part of a Swarm, the following metrics are populated:
| Metric | Description |
| --- | --- |
| **Swarm State** | Numeric status: `0` (Inactive), `1` (Pending), `2` (Active), `3` (Error). |
| **Managers** | Number of Swarm managers. |
| **Nodes** | Total number of nodes in the Swarm. |

---

## Troubleshooting

### Common Issues

**"Permission denied: /var/run/docker.sock" (Linux/macOS)**

* **Cause:** The Lighthouse agent does not have permission to talk to the Docker Daemon.
* **Fix:** Ensure the user running the process is in the `docker` group (see Step 1 above), or run Lighthouse with `sudo`.

**"docker_client_init_error"**

* **Cause:** The agent cannot connect to the Docker API.
* **Fix:** Check if Docker is running (`docker info`). If Docker was just restarted, the agent will automatically attempt to reconnect on the next interval.

**No data appearing**

* Check the agent logs:

```bash
lighthouse --logs "docker-host-01"
```

* If you see metrics like `docker_containers_total`, the connection is successful.
