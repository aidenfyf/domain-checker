# Domain Checker

A single-page tool to check `.com` domain availability. Type one or more business
names separated by commas, press **Check**, and it shows which `.com` domains are
available vs. taken.

## How it works

- Fully static — just `domain_checker.html`. No backend, no API keys, no build step.
- The browser queries Verisign's authoritative `.com` RDAP endpoint
  (`https://rdap.verisign.com/com/v1/domain/<name>.com`) directly:
  - `404` → unregistered → **Available**
  - `200` → registered → **Taken**
- Verisign RDAP sends `Access-Control-Allow-Origin: *`, so the browser call works
  from any deployed origin with no proxy.

## Deploy to Vercel

1. Push this folder to a GitHub repo.
2. In Vercel, **Add New → Project → Import** that repo.
3. Framework preset: **Other**. No build command, no output dir, no env vars.
4. Deploy.

`vercel.json` rewrites `/` to `domain_checker.html`, so the tool loads at the root URL.

## Notes / limits

- `.com` only by design. Adding `.co`/`.ai` would require each TLD's own RDAP base URL.
- Verisign RDAP may rate-limit very large single batches; fine for normal use since
  each visitor queries from their own IP.
