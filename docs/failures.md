# Failure Lab Documentation

This document records the process and learnings from the failure labs in Module 1.

## 1.3.1: Breaking DNS

### Scenario A: Typo in Domain Name
- **Action:** Attempted to resolve a non-existent domain (`my-coll-app.com`).
- **Expected Result:** A DNS-level error.
- **Actual Observed Result (Analysis):**
  The DNS resolution process fails at the TLD (Top-Level Domain) server for `.com`. When asked for the authoritative server for the non-existent domain, the TLD server responds with an `NXDOMAIN` (Non-Existent Domain) error. This is not an HTTP `404` error, because the browser never found an IP address to even send an HTTP request to. The browser displays a connection-level error like "Server not found".

### Scenario C: DNS Record Pointing to Offline Server
- **Action:** Resolved a valid domain name whose `A` record pointed to an IP where no web server was running.
- **Expected Result:** A connection-level error after a successful DNS lookup.
- **Actual Observed Result (Analysis):**
  DNS resolution succeeds perfectly, returning the correct (but outdated) IP address. The failure happens at the next step: the TCP handshake. The browser sends a `SYN` packet to the IP address, but since no server process is listening on the port, the packet is either ignored or rejected. The browser eventually gives up, resulting in an error like "Connection timed out" or "Connection refused". This confirms DNS worked, but the server itself was unreachable.

## 1.3.2: Wrong HTTP Method

- **Action:** Sent a `GET` request to the `/post` endpoint of `httpbin.org`, which expects a `POST`.
- **Expected Result:** A `4xx` client error indicating a method issue.
- **Actual Observed Result (Analysis):**
  The server correctly responded with `405 Method Not Allowed`. This is the proper semantic response. It confirms that the resource (the URL) exists, but the verb (the `GET` method) is not supported for it. This is distinct from a `404 Not Found`. The server also helpfully included an `Allow: POST, OPTIONS` header, indicating which methods are valid.

## 1.3.3: Missing/Altered Headers

- **Action:** Sent a `POST` request with a JSON-like body but without an explicit `Content-Type: application/json` header.
- **Expected Result:** The server would misinterpret the request body.
- **Actual Observed Result (Analysis):**
  The `curl` command automatically added a default header: `Content-Type: application/x-www-form-urlencoded`. The server trusted this header and attempted to parse the JSON-formatted body as form data. Because the body didn't fit the `key=value` format, the server parsed the entire string as a single key with an empty value, placing it in the `form` field of the response. The `json` field was `null`. This demonstrates that servers rely on the `Content-Type` header and do not guess the format of the body.