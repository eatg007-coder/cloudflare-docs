---
pcx_content_type: concept
title: Full setup
sidebar:
  order: 1
  label: About
---

import { FeatureTable, Render } from "~/components";

<Render file="full-setup-definition" product="dns" />

## How to

For more details, refer to [Set up a full domain](/dns/zone-setups/full-setup/setup/).

## Availability

<FeatureTable id="dns.full_setup" />
---
title: Full setup — definition
---

# Full setup (definition)

A **Full setup** is a complete DNS delegation where a domain or subdomain is fully managed through Cloudflare DNS. That means:

- The zone exists in Cloudflare (you can see it in the dashboard or via the API).
- The parent DNS (the registrar or parent zone) delegates the domain/subdomain to Cloudflare by adding the exact Cloudflare nameservers (or by updating the authoritative nameservers for the registered domain).
- DNSSEC and DS records are compatible between the parent and Cloudflare (DNSSEC must be disabled or correctly configured to avoid delegation failures).
- Required records (A, AAAA, CNAME, MX, TXT, etc.) exist inside Cloudflare so the service can function immediately once delegation is in place.

**Why this matters**
If delegation isn't correct at the parent, Cloudflare cannot validate the zone and the dashboard will display statuses such as *Invalid nameservers* or *Pending activation*. Delegation is a two-sided operation: Cloudflare provides authoritative nameservers; the parent must publish the NS delegation for the zone or subdomain.

---

## Key properties

- **Zone type:** Can be a registered domain (example.com) or a delegated subdomain (sub.example.com).
- **Nameserver assignment:** Cloudflare assigns two nameservers per zone (for example `brady.ns.cloudflare.com` and `pat.ns.cloudflare.com`). The parent must publish these exact nameserver hostnames.
- **DNSSEC:** If DNSSEC is enabled at the parent and DS records are not synchronized with Cloudflare, delegation will fail.
- **Conflict avoidance:** An `A` or `CNAME` record at the parent for the same label blocks NS delegation for the label — remove/replace conflicting records when delegating a subdomain.

---

## Quick checklist (tl;dr)

1. Add zone to Cloudflare (dashboard or API). Copy the **two** Cloudflare nameservers.
2. At the parent (registrar or parent zone DNS):
   - For a full domain: set domain nameservers to the two Cloudflare nameservers.
   - For a delegated subdomain: add **NS** records for the subdomain that point to the two Cloudflare nameservers.
3. Remove conflicting records at the parent for the same label (A/CNAME).
4. Disable or synchronize DNSSEC/DS records.
5. Verify with `dig NS <domain> +short` and `dig +trace <domain>`.
6. Once delegation is visible globally, Cloudflare will validate and mark the zone active.

---

If you prefer a deep, practical walkthrough (commands, API examples, troubleshooting) continue to the setup guide: **Set up a full domain**.
---
pcx_content_type: how-to
title: Set up a full domain
sidebar:
  order: 2
  label: Set up a full domain
---

import { Render } from "~/components";

# Set up a full domain

This is the practical, step-by-step guide to put a domain or delegated subdomain fully on Cloudflare. No fluff — follow these steps exactly and your zone will validate.

---

## Before you start — prerequisites

- Access to the Cloudflare account where you will add the zone (e.g. `eatg007@gmail.com`).
- Access to the registrar or the DNS host that controls the parent zone (for example: the domain registrar for example.com, or the DNS host for `uk.com` if delegating `s.uk.com`).
- Knowledge whether you are delegating a root domain (example.com) or a subdomain (s.example.com). The actions differ.
- If DNSSEC is enabled at the parent, be ready to disable it or update DS records to match Cloudflare.

---

## Step 1 — Add the zone to Cloudflare

1. In the Cloudflare dashboard: **Add a site** → enter the domain (or subdomain if creating a zone for a subdomain).
2. Choose the plan and continue. Cloudflare will scan and import existing DNS records (if any).
3. Cloudflare will show **two nameservers assigned to your zone**. Copy them exactly (example: `brady.ns.cloudflare.com`, `pat.ns.cloudflare.com`). **You must use these exact hostnames at the parent.**

**API alternative — create zone via API**

```bash
# Create a zone (replace placeholders)
curl -s -X POST "https://api.cloudflare.com/client/v4/zones" \
  -H "X-Auth-Email: you@example.com" \
  -H "X-Auth-Key: <API_KEY>" \
  -H "Content-Type: application/json" \
  --data '{"name":"example.com","jump_start":true}'
