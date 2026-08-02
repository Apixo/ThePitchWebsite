# Pointing thepitch.com at GitHub Pages

DNS + GitHub setup to serve the site in this repo (`Apixo/ThePitchWebsite`) at
**thepitch.com**. Do the two halves in order: DNS records first, then tell GitHub
about the custom domain.

---

## 1. Add DNS records in GoDaddy

1. Sign in at [godaddy.com](https://godaddy.com) → **My Products** → **thepitch.com**
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
2. Under **Custom domain**, enter **`thepitch.com`** → **Save**.
   This auto-commits a `CNAME` file to the repo — leave it in place.
3. GitHub runs a DNS check; it goes green once the records propagate.
4. Tick **Enforce HTTPS** (may take a few minutes while GitHub provisions the TLS
   certificate). This makes `https://thepitch.com` valid — required for the App
   Store Privacy Policy URL.

---

## After it's live

- `https://thepitch.com/`
- `https://thepitch.com/privacy_policy.html`
- `https://thepitch.com/terms_of_use.html`

`www.thepitch.com` redirects to the apex automatically.

### Verify propagation

```sh
dig +short thepitch.com       # → the four 185.199.x.153 IPs
dig +short www.thepitch.com   # → apixo.github.io, then those IPs
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
