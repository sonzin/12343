# Vietstock Full Page & Multi-tab Screenshot Tool (v4.0)

This artifact documents a Tampermonkey userscript designed to automate the process of capturing high-fidelity screenshots of stock data on the Vietstock finance portal (`finance.vietstock.vn`).

## 🎯 Purpose
Financial analysts often need to archive the state of various data tabs (Overview, Finance, Transactions, etc.) for a specific ticker at a specific point in time. Manually capturing and organizing these is tedious and prone to missing content.

## 🛠️ Technical Features
- **State Persistence & Auto-Resumption**: Uses `GM_setValue(CAPTURE_KEY, 'yes')` to mark a capture session. On each page load, the script checks for this key and automatically triggers the capture routine, ensuring continuity despite the full-page refreshes required by Vietstock's tab navigation.
- **Sequential Immediate Download**: Unlike previous versions that buffered images in memory for a final ZIP archive, v4.0 downloads each screenshot immediately (`link.click()`) after capture. This drastically reduces the browser's memory footprint and prevents data loss if a single tab fails to render.
- **Proactive Hydration**: Triggers a scroll routine (`scrollAll`) to force-load dynamic charts and lazy-loaded tables before rendering the canvas.
- **Sanitization & Hygiene**: 
    - **Modal Purge**: Automatically hides or removes login popups and backdrop overlays that would otherwise obscure data.
    - **UI Transparency**: Self-hides the capture button and status indicators during the rendering phase to ensure a "clean" screenshot.
- **Optimized Rendering**: Employs JPEG compression (`quality: 0.85`) and scaling (`scale: 0.75`) to handle extremely long financial pages without exceeding browser canvas limits.
- **Naming Schema**: Automatically formats filenames as `{SYMBOL}_{INDEX}_{TAB_NAME}.jpg` for organized file explorer sorting.

## 📜 Userscript Implementation
The script adds a floating "📸 Chụp tất cả Tab" button to the bottom right of the Vietstock interface.

```javascript
// ==UserScript==
// @name         Vietstock Screenshot Simple
// @match        https://finance.vietstock.vn/*
// @grant        GM_setValue
// @grant        GM_getValue
// @require      https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js
// ==/UserScript==

// [V4.0 Pattern: autoCapture() runs on DOMContentLoaded if CAPTURE_KEY is active.]
// captureAndDownload().then(goToNextTab);
```

## ⚠️ Known Limitations
- **Navigation Timeouts**: The script waits 2 seconds for content hydration after a page reload. High-latency connections may require longer delays.
- **Session Corruption**: If the user manually navigates away or refreshes during a capture, the `vs_capture_session` in Tampermonkey storage might become desynchronized (requiring a page reload for the "Complete" check).
- **Chart Tainting**: Charts embedded via cross-origin frames (e.g., TradingView or specialized broker widgets) may appear blank due to browser CORS/Taint security policies if not explicitly permitted by the provider.
- **Memory Management**: Despite scaling, capturing 10+ tabs of extremely long pages can still stress available RAM. Use of `image/jpeg` format is mandatory for complex pages.
