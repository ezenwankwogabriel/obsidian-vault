
## Anchora Backend Security — Summary & Action Items

**Setup**

- Domain structure: `anchora.com.ng` (frontend), `api.anchora.com.ng` (backend, NestJS)
- Cloudflare sits in front — same zone (`anchora.com.ng`) covers all proxied subdomains, no separate domain/zone needed for `api.`

**What was found in the logs**

- 961 of 1,000 log lines were 404s from automated internet-wide scanner bots (env file scans, WordPress probes, phpinfo scans, admin panel probes, cloud credential scans) — not a targeted attack, nothing succeeded
- Two notable but still harmless probes: `/api/fs/read?...path=/etc/passwd` (known scanner signature) and `/.git/config`
- 25 hits on `/auth/callback` — worth double-checking if this resembles a real route in your auth flow

**To do**

1. **Confirm `api.anchora.com.ng` DNS record is proxied (orange cloud)** in Cloudflare — this is required for Bot Fight Mode, WAF rules, and rate limiting to apply at all. Check under DNS → Records for the `anchora.com.ng` zone.
2. **Turn on Bot Fight Mode** (or Super Bot Fight Mode on Pro+) — zone-level, under Security → Bots.
3. **Add a WAF custom rule** blocking paths containing `wp-`, `.env`, `phpinfo`, `.git/`, `application_default_credentials` — action: Block. Scope with `http.host eq "app.anchora.com.ng"` if you want it backend-only.
4. **Add a rate-limiting rule** on `api.anchora.com.ng/*` (e.g., >30 req/10s per IP → Challenge/Block), with a tighter rule on sensitive routes like auth/NIN-verification callbacks specifically.
5. **Verify auth guards** on any real routes resembling `/admin`, `/dashboard`, `/account` in your route table.
6. **Add source IP and user-agent to your NestJS request logs** — current logs don't capture these, which limits your ability to distinguish scanner campaigns later.
7. **Re-check logs after Cloudflare changes** — if scanner traffic to `app.` doesn't drop, it usually means the record isn't fully proxied.

Let me know if you want help drafting the actual WAF rule expressions or the rate-limit rule config.