---
sidebar_position: 13
title: Web Analytics
description: Privacy-first website traffic, performance, and interaction tracking.
---

# Web Analytics

This guide explains how to monitor your website traffic using the **Harbor Web Analytics** SDK. This is a lightweight (< 2KB), privacy-first alternative to Google Analytics that integrates directly with your Harbor dashboard.


## Prerequisites

Before starting, ensure you have:

-   Access to the **HTML source code** of the website you want to track.
-   A valid **Harbor ID** and **API Key**.
-   Your website must support HTTPS (recommended) or HTTP.

## How it Works

The Harbor Web SDK is a "fire-and-forget" JavaScript snippet designed for modern performance standards:

1.  **Session Management:** Generates a hashed, anonymous session ID (stored in `sessionStorage`). No PII or persistent cookies are used.
2.  **Batching:** Events are queued and sent in efficient batches using `requestIdleCallback` to ensure **zero impact** on your site's performance (Main Thread blocking is avoided).
3.  **Resilience:** Automatically retries on network failures (5xx errors) but intelligently drops data if authentication fails (4xx errors) to prevent console spam.

---

## Setup Guide

### 1. Standard HTML

Place the following snippet into the `<head>` tag of your website.

```html
<script 
  src="https://cdn.harborscale.com/harbor-web-analytics.js?h=YOUR_HARBOR_ID&k=YOUR_API_KEY]" 
  async 
  defer>
</script>
```

> **Note:** Replace `YOUR_HARBOR_ID` and `YOUR_API_KEY` with your actual credentials.

### 2. Next.js (App Router)

If you are using Next.js, use the `Script` component in your Root Layout (`layout.tsx`) to load the SDK optimally.

```tsx
import Script from 'next/script'

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Script 
          src="https://cdn.harborscale.com/harbor-web-analytics.js?h=YOUR_ID&k=YOUR_KEY"
          strategy="afterInteractive" 
        />
      </body>
    </html>
  )
}

```

### 3. React / Vue (SPA)

The script automatically detects History API changes, so it works with React Router and Vue Router out of the box. You can load it via a hook in your main entry point (e.g., `App.js`).

```jsx
import { useEffect } from 'react';

const HarborAnalytics = () => {
  useEffect(() => {
    const script = document.createElement('script');
    script.src = "https://cdn.harborscale.com/harbor-web-analytics.js?h=YOUR_ID&k=YOUR_KEY";
    script.async = true;
    document.head.appendChild(script);

    return () => {
      // Cleanup on unmount (optional)
      document.head.removeChild(script);
    }
  }, []);

  return null;
};

```

---

## Configuration

You can enable or disable specific tracking modules by adding query parameters to the script URL.

| Parameter | Description | Default |
| --- | --- | --- |
| `h` | **Required.** Your Harbor ID. | - |
| `k` | **Required.** Your API Key. | - |
| `track-perf` | Monitor Core Web Vitals (LCP, CLS, FID). | `true` |
| `track-scroll` | Track scroll depth milestones (25%, 50%, etc). | `true` |
| `track-clicks` | Track clicks on buttons, links, and outbound URLs. | `true` |
| `track-errors` | Capture JavaScript exceptions and crashes. | `true` |
| `batch-size` | Number of events to queue before sending. | `50` |
| `debug` | Enable verbose console logging for development. | `false` |

**Example: specific configuration**

```html
<script src="...?h=1&k=xyz&track-scroll=false&batch-size=20"></script>
```

---

## Captured Metrics

The SDK automatically pushes the following `cargo_id`s to your dashboard.

### 1. Session & Identity

| Cargo ID | Trigger | Description |
| --- | --- | --- |
| `session.start` | Script Load | Fired once when a new session is initialized. |
| `pageview` | Page Load / Route | Fired on every page load. Supports SPA navigation. |
| `page.time_on_page` | Route Change | Duration (seconds) spent on the *previous* page before navigating. |

### 2. Marketing (UTM Parameters)

*Only sent if standard UTM parameters are detected in the URL.*

| Cargo ID | Description |
| --- | --- |
| `marketing.session_start` | Fired if **any** UTM parameter is present on landing. |
| `marketing.source` | Value of `utm_source` (e.g., "google"). |
| `marketing.medium` | Value of `utm_medium` (e.g., "cpc"). |
| `marketing.campaign` | Value of `utm_campaign`. |

### 3. Device Context

*Captured once per session when the browser is idle.*

| Cargo ID | Description |
| --- | --- |
| `device.screen_width` | Total width of the user's monitor. |
| `device.viewport_width` | Width of the actual browser window. |
| `device.pixel_ratio` | Screen density (e.g., `2` for Retina). |
| `device.conn_type` | Network type (e.g., `4g`, `wifi`) if supported by browser. |

### 4. User Interaction

*Requires `track-clicks=true` and `track-scroll=true`.*

| Cargo ID | Trigger | Description |
| --- | --- | --- |
| `click.button` | Click | User clicked a `<button>` element. |
| `click.a` | Click | User clicked a link (`<a>`). |
| `click.input` | Click | User clicked an input (e.g., submit button). |
| `click.outbound` | Click | Fired when clicking a link to a **different domain**. |
| `scroll.depth_25` | Scroll | User scrolled past 25% of the page. |
| `scroll.depth_50` | Scroll | User scrolled past 50% of the page. |
| `scroll.depth_75` | Scroll | User scrolled past 75% of the page. |
| `scroll.depth_100` | Scroll | User reached the bottom of the page. |

### 5. Performance (Core Web Vitals)

*Requires `track-perf=true`.*

| Cargo ID | Unit | Description |
| --- | --- | --- |
| `perf.lcp` | ms | **Largest Contentful Paint**: Time for main content to load. |
| `perf.fid` | ms | **First Input Delay**: Time from tap to browser response. |
| `perf.cls` | score x1000 | **Cumulative Layout Shift**: Visual stability score. |

### 6. Reliability

*Requires `track-errors=true`.*

| Cargo ID | Trigger | Description |
| --- | --- | --- |
| `error.js` | Exception | Uncaught JavaScript errors. |

---

## Custom Events (API)

The script exposes a global object `window.harbor` that allows you to track custom events within your application logic.

### `harbor.track(cargoId, value, suffix)`

* **cargoId** (string): The name of the metric.
* **value** (number): The value to record (must be a number).
* **suffix** (string, optional): Appends a string to the `ship_id` for segmentation.

**Example: Tracking a "Sign Up" button click**

```javascript
// On button click
if (window.harbor) {
  window.harbor.track('signup_click', 1);
}
```

**Example: Tracking Cart Value**

```javascript
// When user adds item to cart
window.harbor.track('cart_value', 49.99, 'cart_update');
```

---

## Troubleshooting

### Common Issues

**No data appearing in Dashboard**

* **Ad Blockers:** Some strict ad blockers may block the script.
* **Queueing:** The script batches events to save battery/data. Wait ~10 seconds or navigate away from the page (triggering a beacon flush) to see data.
* **Console Errors:** Open Developer Tools (F12) and check the Console tab.

**403 Forbidden Error**

* **Cause:** The API Key provided in the `k` parameter is invalid or does not match the Harbor ID.
* **Fix:** Regenerate your key in Harbor Settings and update the script tag.

**Content Security Policy (CSP)**

* If your site uses CSP, you must allow the Harbor CDN and Ingest Endpoint.

```http
Content-Security-Policy: script-src 'self' https://cdn.harborscale.com]; connect-src 'self' https://harborscale.com;
```
