# Continu Reporting API — Partner Portal

This is the source for a private, gated Mintlify docs site you'll share with a customer who needs the Continu Reporting API.

**Total setup time:** ~90 minutes including DNS propagation
**Total cost:** $0/month (Mintlify Hobby + Cloudflare Access free tier)

---

## What's in this repo

```
mintlify-portal/
├── docs.json                    # Mintlify config (navigation, theming, OpenAPI ref)
├── openapi.json                 # OpenAPI 3.0 spec — drives the API reference tab
├── introduction/
│   ├── welcome.mdx              # Landing page
│   └── quickstart.mdx           # First-request walkthrough
├── api-reference/
│   ├── authentication.mdx
│   ├── request-model.mdx        # The field-selection pattern explained
│   ├── date-handling.mdx
│   ├── rate-limits.mdx
│   ├── error-handling.mdx
│   └── sync-patterns.mdx        # Warehouse load patterns
└── downloads/
    ├── index.mdx                # Download index page
    ├── Continu-Reporting-API-Integration-Guide.docx
    ├── Continu-Reporting-API.postman_collection.json
    ├── Continu-Reporting-API-spec.json    # Swagger 2.0
    └── openapi.json                       # OpenAPI 3.0 (copy)
```

---

## Deployment

### Step 1: Create a GitHub repo (5 min)

1. Create a new **private** repository on GitHub. Suggested name: `continu-partner-docs` or `reports-api-portal`.
2. Clone it locally and copy the contents of this folder into it.
3. Commit and push to `main`.

```bash
git init
git add .
git commit -m "Initial Mintlify portal"
git remote add origin git@github.com:continu/continu-partner-docs.git
git push -u origin main
```

### Step 2: Sign up for Mintlify Hobby (5 min)

1. Go to <https://mintlify.com> and click **Get started**.
2. Sign up with the GitHub account that owns the docs repo.
3. Pick the **Hobby** (free) plan.
4. During onboarding, point Mintlify at the GitHub repo you just created.
5. Install the Mintlify GitHub App when prompted — this is what lets Mintlify auto-deploy on every push.

Within ~2 minutes, Mintlify will deploy the site to a default subdomain like `continu-partner-docs.mintlify.app`. Verify it loads — you should see the landing page, all the MDX pages in the sidebar, and the API reference tab with all 14 endpoints.

### Step 3: Point a custom domain at it (10 min + DNS propagation)

In Mintlify's dashboard:

1. Go to **Settings → Custom Domain**.
2. Enter `reporting.continu.com`.
3. Mintlify will show you a CNAME record to add to your DNS.

In your DNS provider (Cloudflare, Route 53, etc.):

1. Add the CNAME record Mintlify gave you.
2. Wait 5-30 minutes for DNS to propagate.

Once propagated, your docs are live at the custom domain. **At this point the site is publicly accessible** — anyone with the URL can read it. Next step locks it down.

### Step 4: Put Cloudflare Access in front (30 min)

This is the gating step. Free for up to 50 users.

#### 4a. Make sure your domain is on Cloudflare

If `continu.com` isn't already on Cloudflare, move DNS there first. (If it is, skip ahead.) This is a one-time setup that takes 24 hours to fully propagate but works immediately for new subdomains.

#### 4b. Enable Zero Trust

1. In the Cloudflare dashboard, go to **Zero Trust** (left sidebar).
2. On first use, you'll be asked to set up a Zero Trust team — pick a team name (e.g., `continu`).
3. Select the **Free** plan ($0, up to 50 users).

#### 4c. Create an Access application

1. In Zero Trust, go to **Access → Applications**.
2. Click **Add an application → Self-hosted**.
3. Configure:
   - **Application name:** `Reports API Partner Docs`
   - **Session duration:** 24 hours (reasonable default)
   - **Application domain:** the subdomain you set up (e.g., `reporting.continu.com`)
4. Click **Next**.

#### 4d. Create an Access policy

1. **Policy name:** `Customer access — <CustomerName>`
2. **Action:** Allow
3. **Include:** Add the rule **Emails** and list the customer's engineering contacts:
   - `engineer1@customer.com`
   - `engineer2@customer.com`
   - (etc.)
4. Optionally add internal Continu emails so your team can access for QA.
5. Save.

#### 4e. Configure the login flow

1. **Identity providers:** Cloudflare's default **One-time PIN** flow works without any SSO setup — visitors enter their email and Cloudflare emails them a 6-digit code to verify. Zero setup, works immediately.
2. (Optional, later) Add Google/Okta/Azure AD if you want SSO instead of email codes.

#### 4f. Test it

1. Open an incognito window and visit your subdomain.
2. You should see Cloudflare's login screen, not your docs.
3. Enter an email on your allowlist → receive a PIN → enter it → land on the docs.
4. Try with a non-allowlisted email — access should be denied.

### Step 5: Share with the customer

Email the customer's technical lead with:

- The portal URL (e.g., `https://reporting.continu.com`)
- The list of email addresses you've authorized for access
- A note that access is via Cloudflare-issued PIN to the listed emails
- Your support contact for questions

Sample text:

> Hi [Name],
>
> Here's access to the Continu Reporting API documentation for your team's eval:
>
> **Portal:** https://reporting.continu.com
> **Access:** I've authorized [list emails]. They'll be prompted for a one-time code sent to their email on first visit.
>
> Inside the portal you'll find:
> - A walkthrough guide (auth, request model, date handling, sync patterns)
> - The full API reference (all 14 endpoints with try-it-out)
> - Downloads: Word integration guide, Postman collection, OpenAPI spec
>
> Tokens for the API itself will come separately under [NDA reference]. For questions, reach out to [your contact] directly.
>
> Best,
> [Your name]

---

## Maintenance

### Updating content

The docs are docs-as-code — push to `main` and Mintlify rebuilds within 30-60 seconds.

For local preview before pushing, install Mintlify's CLI:

```bash
npm install -g mintlify
cd mintlify-portal
mintlify dev
```

This serves the docs at `http://localhost:3000` with live reload.

### Updating the API spec

The `openapi.json` file at the repo root drives the API reference tab. Replace it with a newer version and push — the reference updates automatically.

If you regenerate `openapi.json`, also copy it to `downloads/openapi.json` so the download link stays current:

```bash
cp openapi.json downloads/openapi.json
git commit -am "Update API spec"
git push
```

### Revoking customer access

When the eval wraps:

1. Cloudflare Zero Trust → **Access → Applications**
2. Open the application
3. Remove emails from the policy, or delete the policy entirely
4. Save

The next time a customer visitor tries to access the URL, they'll be denied at the gate.

---

## Costs (recap)

| Component | Cost |
|---|---|
| Mintlify Hobby | $0/month |
| Custom domain (Mintlify) | included in Hobby |
| Cloudflare Access (≤50 users) | $0/month |
| DNS (assuming you already have Cloudflare) | $0/month |
| **Total** | **$0/month** |

---

## Common pitfalls

**Don't push the API token into the repo.** The docs reference `<your-token>` placeholders throughout — keep it that way. Tokens go through Cloudflare Access-protected channels, not in markdown.

**Don't make the GitHub repo public.** Even though the docs are gated behind Cloudflare Access at runtime, the GitHub repo is the source. Keep it private. Mintlify reads it via the GitHub App — public is not required.

**Don't skip the NDA confirmation step.** Cloudflare Access is access control, not an NDA. Make sure your existing NDA with the customer covers technical materials before sending the URL.

**Don't share the access PIN.** Each authorized email gets their own PIN; they should not forward it to colleagues. If a colleague needs access, add their email to the policy instead.
