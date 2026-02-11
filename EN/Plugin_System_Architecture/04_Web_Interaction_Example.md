# Plugin Example: Web Interaction

This document outlines the design of plugins for interacting with the Web, including Webhooks and browser control.

---

### 1. Webhook

Webhook capabilities are implemented through `Listener` plugins, enabling the Agent to passively receive events from the Web.

*   **Plugin Type:** `listener`
*   **Responsibility:** Starts an HTTP service to listen on a specific URL endpoint and forwards received requests to the Core's event queue.
*   **`plugin.json` Example:**
    ```json
    {
      "name": "Webhook-Listener",
      "type": "listener",
      "entryPoint": "./WebhookListener.js",
      "config": {
        "port": 8080,
        "endpoint": "/hooks/github"
      }
    }
    ```
*   **Implementation Strategy:**
    1.  The `start(eventQueue, config)` method of `WebhookListener.js` is invoked by the Core.
    2.  Within the `start` method, an HTTP service is launched on the specified `port` using a framework like `Express` or `Fastify`.
    3.  The service listens for `POST` or `GET` requests on the configured `endpoint`.
    4.  Upon receiving a request, information such as `headers`, `body`, and `query_params` is packaged into a standard internal event object.
    5.  `eventQueue.push({ source: 'webhook', type: 'http_request', payload: ... })` is called to push the event into the queue.

---

### 2. Browser Control

Browser control is a powerful suite of `Tool` plugins that allow the LLM to "see" and "operate" web pages much like a human would.

*   **Plugin Type:** `tool` (usually a suite containing multiple tools).
*   **Core Dependency:** A backend **Browser Manager** that controls one or more headless browser instances via `Puppeteer` or `Playwright`. For security reasons, the browser instances themselves should run within a sandboxed environment.
*   **`plugin.json` Example:**
    ```json
    {
      "name": "Browser-Control-Suite",
      "type": "tool",
      "entryPoint": "./BrowserTools.js"
    }
    ```
*   **Tool Examples (`BrowserTools.js` exports multiple tools):**
    *   `browser:new_page(url)`: Opens a new page and navigates to the specified URL, returning a `page_id`.
    *   `browser:scrape(page_id, selectors, format)`: Extracts content from a specified page based on CSS `selectors` and can return it in `text` or `markdown` format.
    *   `browser:click(page_id, selector)`: Clicks an element on the specified page.
    *   `browser:type(page_id, selector, text)`: Enters text into an input field on the specified page.
    *   `browser:screenshot(page_id)`: Takes a screenshot of the page and returns the Base64-encoded image data.
*   **Implementation Strategy:**
    *   The `execute` method of each tool performs the corresponding `Playwright` operation through the backend "Browser Manager."
    *   The manager maintains the mapping between `page_id`s and the actual page objects.
    *   Results from `scrape` or image data from `screenshot` are returned to the LLM, enabling it to "see" web content and decide on subsequent actions.
