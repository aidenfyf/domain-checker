# Domain Checker — Build Brief

## Problem to Solve

A client-facing domain availability checker exists as a working HTML file but the core lookup functionality is broken. The task is to wire up a reliable backend that the frontend can call to check live domain availability. The frontend is complete and should not need significant changes — the fix lives entirely on the backend.

---

## What Exists

A single self-contained HTML file (`domain_checker.html`) with:

- Input field accepting comma-separated business names
- Slug conversion logic that appends `.com` to each name
- Animated red-to-yellow morphing orb loading state
- Results UI split into Available / Taken sections
- Instructions displayed above the input field
- `.com` only — no `.co`, no `.ai`

---

## What Is Broken

The domain lookup. Every browser-based approach has been attempted and failed:

- **Vercel MCP** — requires authenticated server-side session; tool calls fire but results never return to the browser
- **RDAP (rdap.org)** — returns `200` for both taken and available domains, results are unreliable
- **GoDaddy MCP (`api.godaddy.com/v1/domains/mcp`)** — same hanging problem as Vercel MCP; never completes in a browser context

**Root cause:** All reliable domain availability APIs either require a server-side API key or authenticated MCP sessions that cannot be established from a browser artifact. The current frontend fetch calls have nowhere reliable to go.

---

## What Needs to Be Built

A lightweight backend — Vercel serverless function, Cloudflare Worker, or equivalent — that acts as a proxy between the frontend and a domain availability API.

### Backend requirements

- Accepts a POST request with a JSON body: `{ "domains": ["example.com", "another.com"] }`
- Holds the API key server-side (environment variable, never exposed to the browser)
- Queries domain availability for each domain
- Returns clean JSON: `{ "available": ["example.com"], "taken": ["another.com"] }`
- Handles CORS so the HTML file can call it from any origin

### Recommended API

**GoDaddy**
- Endpoint: `https://api.godaddy.com/v1/domains/available?domain=example.com`
- Auth header: `Authorization: sso-key {key}:{secret}`
- Free tier available at [developer.godaddy.com](https://developer.godaddy.com)

**Fallback option**
WhoisXML API — `https://domain-availability.whoisxmlapi.com/api/v1?apiKey={key}&domainName=example.com&credits=DA` — 500 free credits on signup, no card required.

---

## Frontend Integration

Once the backend endpoint is live, update the fetch call in `domain_checker.html` to point at it. The current frontend already handles:

- Parsing the response into available/taken arrays
- Rendering results into the UI
- Showing/hiding the loading orb

The only change needed is replacing the broken lookup logic with a clean `fetch('YOUR_ENDPOINT', { method: 'POST', body: JSON.stringify({ domains }) })` call.

---

## Design Reference

| Token | Value |
|---|---|
| Background | `#0a0a0a` |
| Surface | `#111111` |
| Accent green | `#00e5a0` |
| Heading font | Syne |
| Body font | DM Sans |
| Mono font | DM Mono |
| Loading state | Red-to-yellow morphing orb with border-radius animation |
