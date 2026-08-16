# Pointing thepitchme.com at GitHub Pages

DNS + GitHub setup to serve the site in this repo (`Apixo/ThePitchWebsite`) at
**thepitchme.com**. Do the two halves in order: DNS records first, then tell GitHub
about the custom domain.

---

## 1. Add DNS records in GoDaddy

1. Sign in at [godaddy.com](https://godaddy.com) → **My Products** → **thepitchme.com**
   → **DNS** (Manage DNS).
2. In **Records**, first **delete any existing `A` record on host `@`** (GoDaddy
   ships a default parked A record — remove it or it will conflict).
3. Add the records below.

### Apex domain — four A records (host `@`)

| Type | Host | Value             | TTL    |
|------|------|-------------------|--------|
| A    | `@`  | `185.199.108.153` | 1 Hour |
| A    | `@`  | `185.199.109.153` | 1 Hour |
| A    | `@`  | `185.199.110.153` | 1 Hour |
| A    | `@`  | `185.199.111.153` | 1 Hour |

### www subdomain — one CNAME

| Type  | Host  | Value              | TTL    |
|-------|-------|--------------------|--------|
| CNAME | `www` | `apixo.github.io`  | 1 Hour |

> The CNAME value is `apixo.github.io` — **not** `apixo.github.io/ThePitchWebsite`,
> and **no** trailing dot in GoDaddy's field.

### Optional — four AAAA records for IPv6 (host `@`)

Helps some networks reach the site faster.

```
2606:50c0:8000::153
2606:50c0:8001::153
2606:50c0:8002::153
2606:50c0:8003::153
```

4. **Save.** Changes apply in minutes; global propagation can take up to an hour
   (occasionally longer).

---

## 2. Set the custom domain in GitHub Pages

Requires Pages to be enabled first (Settings → Pages → Deploy from branch → `main`
/ root).

1. Go to **https://github.com/Apixo/ThePitchWebsite/settings/pages**.
2. Under **Custom domain**, enter **`thepitchme.com`** → **Save**.
   This auto-commits a `CNAME` file to the repo — leave it in place.
3. GitHub runs a DNS check; it goes green once the records propagate.
4. Tick **Enforce HTTPS** (may take a few minutes while GitHub provisions the TLS
   certificate). This makes `https://thepitchme.com` valid — required for the App
   Store Privacy Policy URL.

---

## After it's live

- `https://thepitchme.com/`
- `https://thepitchme.com/privacy_policy.html`
- `https://thepitchme.com/terms_of_use.html`

`www.thepitchme.com` redirects to the apex automatically.

---

## Universal Links (`apple-app-site-association`)

`.well-known/apple-app-site-association` in this repo pairs with the
`applinks:thepitchme.com` entitlement in the iOS app, so
`https://thepitchme.com/apply/{employer_id}` opens the employer storefront
in-app. `.nojekyll` sits alongside it because Jekyll strips dot-directories
from the published site — without it, `.well-known/` never deploys.

**Verified working 2026-08-16 — don't "fix" the content type.** GitHub Pages
serves this file as `application/octet-stream` (it types by file extension and
there isn't one), which reads like a violation of Apple's `application/json`
requirement. It isn't a problem in practice: iOS doesn't fetch the origin, it
fetches Apple's CDN, and the CDN ingested the file happily and re-serves it as
proper JSON. Check the CDN, not the origin:

```sh
curl -s https://app-site-association.cdn-apple.com/a/v1/thepitchme.com
```

A 200 with the file contents means Universal Links are good. A 404 means Apple
hasn't ingested it — give it a few hours after a change, and only then suspect
the content type (the fix would be a host that allows custom headers, e.g.
Cloudflare Pages or Netlify via a `_headers` file).

Note the CDN 404s for `www.thepitchme.com`, because the entitlement only claims
the apex. Links the app generates are apex, so this is fine; if a `www` link
ever needs to open in-app, add `applinks:www.thepitchme.com` to
`ThePitch.entitlements` and serve the AASA there too.

Two things still missing before the link fully works: the `/apply/{id}` page
on this site (it currently 404s for anyone without the app), and the
Associated Domains capability enabled on the App ID in the developer portal.

### Verify propagation

```sh
dig +short thepitchme.com       # → the four 185.199.x.153 IPs
dig +short www.thepitchme.com   # → apixo.github.io, then those IPs
```

---

## Reference — GitHub Pages apex IPs

These are GitHub's published Pages IPs (current as of 2026-08). If GitHub ever
changes them, use the values in their docs:
<https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site>

```
A     185.199.108.153
A     185.199.109.153
A     185.199.110.153
A     185.199.111.153
AAAA  2606:50c0:8000::153
AAAA  2606:50c0:8001::153
AAAA  2606:50c0:8002::153
AAAA  2606:50c0:8003::153
CNAME (www) apixo.github.io
```
