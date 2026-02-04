---
sidebar_position: 12
title: HTTP Uptime Monitoring
description: Monitor website availability, response codes, and latency with Harbor Lighthouse.
---

# HTTP Uptime Monitoring

This guide explains how to use **Harbor Lighthouse** to monitor the availability and performance of your websites or APIs. The `uptime` collector acts as a persistent probe, pinging your target URLs to track uptime history, response times, and status codes.

**_Repo Link:_** https://github.com/harborscale/harbor-lighthouse

## Prerequisites

Before starting, ensure you have:

-   **Harbor Lighthouse** installed (See [Installation Guide](/docs/lighthouse/)).
-   A **Target URL** (e.g., `https://google.com` or an internal API endpoint).
-   **Harbor Scale account** credentials.

## How it Works

The Lighthouse agent performs periodic HTTP/HTTPS `GET` requests to the specified target. It captures the standard uptime status and breaks down the request lifecycle into granular timing metrics (DNS lookup, TCP connection, TLS handshake, etc.).

This is useful for:
* **External Monitoring:** Verifying your public website is accessible and checking CDN performance.
* **Internal Monitoring:** Checking if internal APIs or dashboards are running inside your private network.
* **Performance Debugging:** Identifying if slowness is caused by DNS resolution, SSL handshakes, or backend processing (TTFB).

---

## Setup Guide

### 1. Add the Monitor

Use the `lighthouse` CLI to create a new uptime monitor. You must provide the `target_url` parameter.

**For Harbor Scale Cloud (Linux/macOS):**

```bash
sudo lighthouse --add --name "my-website" --harbor-id "YOUR_HARBOR_ID" --key "YOUR_API_KEY" --source uptime --param target_url="https://harborscale.com"
```

**For Harbor Scale Cloud (Windows):**

```powershell
lighthouse --add --name "my-website" --harbor-id "YOUR_HARBOR_ID" --key "YOUR_API_KEY" --source uptime --param target_url="https://harborscale.com"
```

**For Self-Hosted / OSS (Linux/macOS):**

```bash
sudo lighthouse --add --name "internal-api" --endpoint "http://YOUR_IP:8000" --key "YOUR_API_KEY" --source uptime --param target_url="http://192.168.1.50:3000/health"
```

**For Self-Hosted / OSS (Windows):**

```powershell
lighthouse --add --name "internal-api" --endpoint "http://YOUR_IP:8000" --key "YOUR_API_KEY" --source uptime --param target_url="http://192.168.1.50:3000/health"
```

> **Note:** The `--name` you choose will identify this specific check in your dashboard.

---

## Configuration Parameters

The `uptime` collector uses the `--param` flag to pass specific settings. You can chain multiple params if needed.

| Parameter | Required? | Description | Default |
| --- | --- | --- | --- |
| `target_url` | ✅ Yes | The full URL to monitor (must include `http://` or `https://`). | - |
| `timeout_ms` | ❌ No | How long to wait for a response before marking it as "Down" (in milliseconds). | `10000` (10s) |

### Example: Custom Timeout

To monitor a slow internal service and extend the timeout to 20 seconds:

**Linux/macOS:**

```bash
sudo lighthouse --add --name "slow-service" --harbor-id "123" --key "xyz" --source uptime --param target_url="http://local-service" --param timeout_ms=20000
```

---

## Available Metrics

The collector reports the following data points for every check:

### General Status

| Metric | Description |
| --- | --- |
| **Status** | The health state (`up` or `down`). |
| **Status Code** | The HTTP response code received (e.g., `200`, `404`, `503`). |
| **Total Latency** | Total time taken for the request to complete (in milliseconds). |
| **Response Size** | Size of the response body in bytes. |
| **Redirects** | Number of redirects followed during the request. |
| **Cache Hit** | Detected CDN cache status (`1` for HIT, `0` for MISS/Other). Supports Vercel, Cloudflare, Fastly, Netlify, and generic `x-cache`. |

### Detailed Timings (Waterfalls)

The collector breaks down latency into specific phases to help diagnose bottlenecks:

| Metric | Description |
| --- | --- |
| **DNS Lookup** | Time taken to resolve the domain name. |
| **TCP Connect** | Time taken to establish the TCP connection. |
| **TLS Handshake** | Time spent negotiating the SSL/TLS secure connection. |
| **TTFB** | **Time To First Byte**. Time between the request being sent and the first byte of the response being received (indicates backend processing speed). |
| **Download Time** | Time taken to download the response body after the first byte. |

---

## Troubleshooting

### Common Issues

**"Monitor is Unhealthy" or "Connection Refused"**

* **Cause:** The machine running Lighthouse cannot reach the `target_url`.
* **Fix:** Try running `curl -I <url>` (Linux/Mac) or `Invoke-WebRequest -Uri <url> -Method Head` (Windows) from the terminal on the same machine to verify connectivity.

**"Timeout" errors on valid sites**

* **Cause:** The server is responding too slowly for the default 10-second limit.
* **Fix:** Increase the timeout using `--param timeout_ms=20000`.

**SSL/TLS Errors**

* **Cause:** Lighthouse validates SSL certificates by default. If your internal site uses a self-signed certificate, the check may fail.
* **Fix:** Ensure the machine running Lighthouse trusts the CA of your internal certificate.
