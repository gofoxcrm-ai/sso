# Gofox Single Sign-On (SSO)

Enterprise login for Gofox tenants via **OIDC** and **SAML 2.0** — same product category as EngageBay’s [SSO](https://www.engagebay.com/api).

> Sibling products: [REST API](https://github.com/gofoxcrm-ai/restapi) · [Tracking Code API](https://github.com/gofoxcrm-ai/trackingcodeapi) · [Webhooks](https://github.com/gofoxcrm-ai/webhooks)

---

## Status

| Area | Status |
|------|--------|
| OIDC authorization code | ✅ Live |
| SAML SP-initiated ACS | ✅ Live |
| Email / domain discovery | ✅ Live |
| SCIM provisioning | 🚧 Not yet |
| Screenshots / IdP setup video | 🚧 Placeholders below |

Plan entitlement: **`sso`** (Prime+).

---

## Base URL

```
Production: https://api.gofox.io/api/v1/public/sso
Local:      http://localhost:4000/api/v1/public/sso
```

Client app login lives on your tenant host (e.g. `https://app.gofox.io`).

---

## Configure SSO

1. Gofox: **Account Settings → SSO** (or Security / SSO)
2. Choose provider type: **OIDC** or **SAML**
3. Enter issuer, client id/secret (OIDC) or IdP metadata / cert (SAML)
4. Optionally enforce SSO for email domains
5. Save — users with matching domains can discover SSO on the login page

<!-- SCREENSHOT: docs/assets/sso-settings.png
     Placeholder — SSO settings form (OIDC/SAML fields).
-->

![SSO settings (placeholder)](docs/assets/sso-settings.png)

<!-- VIDEO: docs/assets/sso-okta-setup.mp4
     Placeholder — configure Okta/Azure AD → login with SSO in Gofox.
-->

[IdP setup video (placeholder)](docs/assets/sso-okta-setup.mp4)

---

## Public endpoints (working)

| Method | Path | Description |
|--------|------|-------------|
| `GET` / `POST` | `/sso/discover` | Resolve org + SSO provider from email / domain |
| `GET` | `/sso/:orgSlug/oidc/login` | Start OIDC login |
| `GET` | `/sso/:orgSlug/oidc/callback` | OIDC callback |
| `GET` | `/sso/:orgSlug/saml/login` | Start SAML login |
| `POST` | `/sso/:orgSlug/saml/callback` | SAML ACS |
| `GET` | `/sso/:orgSlug/metadata` | SP metadata (SAML) |

Implementation: `gofox-server/src/modules/auth/public-sso.routes.ts` + `gofox-server/src/lib/sso.ts`.

### Discover

```bash
curl -s -X POST "$API/sso/discover" \
  -H "Content-Type: application/json" \
  -d '{"email":"user@acme.com"}'
```

Typical response includes `loginUrl` pointing at OIDC or SAML login for that org.

---

## OIDC checklist

| Item | Notes |
|------|-------|
| Redirect URI | `{API_BASE_URL}/api/v1/public/sso/{orgSlug}/oidc/callback` |
| Scopes | `openid email profile` (minimum) |
| Client auth | Client secret stored encrypted server-side |

## SAML checklist

| Item | Notes |
|------|-------|
| ACS URL | `{API_BASE_URL}/api/v1/public/sso/{orgSlug}/saml/callback` |
| Entity ID / metadata | `GET .../sso/{orgSlug}/metadata` |
| NameID | Prefer email |
| Signing cert | Paste IdP X.509 in Gofox settings |

---

## Login UX

1. User enters work email on Gofox login
2. Client calls `/sso/discover`
3. If SSO configured → redirect to IdP
4. Callback issues Gofox session cookies / tokens
5. Optional: enforce SSO (block password) for matched domains; admins may keep password fallback

---

## Adding capabilities

| Feature | Where to extend |
|---------|-----------------|
| Extra OIDC claims → role mapping | `lib/sso.ts` JIT user provisioning |
| IdP-initiated SAML | New route + relay state handling |
| SCIM | New module under `/api/v1/scim/v2` (future) |

Document new routes in this README when they ship.

---

## Media placeholders

| File | Purpose |
|------|---------|
| `docs/assets/sso-settings.png` | Tenant SSO settings |
| `docs/assets/sso-login-discover.png` | Login discover / Continue with SSO |
| `docs/assets/sso-okta-setup.mp4` | Okta / Entra walkthrough |

---

## Security notes

- Never expose IdP client secrets to the browser
- Validate ACS signatures and audience on every assertion
- Prefer short-lived state / nonce for OIDC
- Rotate IdP certificates before expiry

---

## License

Documentation © Gofox. SSO subject to plan entitlements and your IdP agreement.
