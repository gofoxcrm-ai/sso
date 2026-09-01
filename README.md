# Gofox Single Sign-On (SSO)

Enterprise login for Gofox tenants via **OIDC** and **SAML 2.0**.

> Sibling products: [REST API](https://github.com/gofoxcrm-ai/restapi) · [Tracking Code API](https://github.com/gofoxcrm-ai/trackingcodeapi) · [Webhooks](https://github.com/gofoxcrm-ai/webhooks)

**Phased docs:** [PHASES.md](./PHASES.md)

---

## Status overview

| Phase | Area | Status |
|-------|------|--------|
| 1 | OIDC + SAML SP-initiated | ✅ Live |
| 2 | Discover, domain enforcement, admin fallback | ✅ Live |
| 3 | Social Google/LinkedIn; SCIM / JIT | Social ✅ · SCIM/JIT ❌ |
| Media | Screenshots / video | 🚧 Placeholders |

Plan entitlement: **`sso`** (Prime+).

---

## Base URL

```
Production: https://api.gofox.io/api/v1/public/sso
Local:      http://localhost:4000/api/v1/public/sso
```

Client app login lives on your tenant host (e.g. `https://app.gofox.io`).

---

# Phase 1 — Enterprise SSO (SAML / OIDC)

## Configure SSO

1. Gofox: **Account Settings → SSO**
2. Choose provider type: **OIDC** or **SAML**
3. Enter issuer, client id/secret (OIDC) or IdP entry point / cert / metadata (SAML)
4. Optionally list **enforce email domains**
5. Optionally allow **password fallback for admins**
6. Save — matching users can discover SSO on the login page

Settings fields (as stored): `enabled`, `provider` (`saml`|`oidc`), `issuerUrl`, `clientId`, `clientSecret`, `metadataUrl`, `domainHint`, `enforceEmailDomains[]`, `allowPasswordFallbackForAdmins`, `samlEntryPoint`, `samlIdpCert`, `notes`.

![SSO settings (placeholder)](docs/assets/sso-settings.png)

[IdP setup video (placeholder)](docs/assets/sso-okta-setup.mp4)

## Public endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` / `POST` | `/discover` | Resolve org + SSO from email / domain |
| `GET` | `/:orgSlug/oidc/login` | Start OIDC login |
| `GET` | `/:orgSlug/oidc/callback` | OIDC callback (redirect URI) |
| `GET` | `/:orgSlug/saml/login` | Start SAML login |
| `POST` | `/:orgSlug/saml/callback` | SAML ACS |
| `GET` | `/:orgSlug/metadata` | SP metadata XML **and Entity ID** |

Full paths are under `/api/v1/public/sso/...`.

Implementation: `gofox-server/src/modules/auth/public-sso.routes.ts` + `gofox-server/src/lib/sso.ts`.

## OIDC checklist

| Item | Value |
|------|-------|
| Redirect URI | `{API_BASE_URL}/api/v1/public/sso/{orgSlug}/oidc/callback` |
| Scopes | `openid email profile` (minimum) |
| Client auth | Client secret stored encrypted server-side |

## SAML checklist

| Item | Value |
|------|-------|
| ACS URL | `{API_BASE_URL}/api/v1/public/sso/{orgSlug}/saml/callback` |
| Entity ID / metadata | `{API_BASE_URL}/api/v1/public/sso/{orgSlug}/metadata` |
| NameID | Prefer email |
| Signing cert | Paste IdP X.509 in Gofox settings |

> **Important:** Use the **`/metadata`** URL as the SP Entity ID (matches server). Do not invent a `/saml` Entity ID path unless the product UI is updated to match.

---

# Phase 2 — Login UX & policy

## Discover

```bash
curl -s -X POST "https://api.gofox.io/api/v1/public/sso/discover" \
  -H "Content-Type: application/json" \
  -d '{"email":"user@acme.com"}'
```

Typical response includes `loginUrl` pointing at OIDC or SAML login for that org. The Gofox login page also supports discover via query (`?email=`).

## Login flow

1. User enters work email on Gofox login  
2. Client calls `/sso/discover`  
3. If SSO configured → redirect to IdP  
4. Callback validates assertion/tokens → `completeSsoLogin`  
5. Session cookie + redirect to `{CLIENT_URL}/auth/callback?token=…&next=/home`

## Enforcement

| Rule | Behavior |
|------|----------|
| Matching enforce domain | Password login blocked (`SSO_REQUIRED`) |
| Matching enforce domain | Google/LinkedIn social login blocked |
| Self-registration | Blocked for SSO domains (`SSO_REGISTRATION_BLOCKED`) |
| Admin fallback | If `allowPasswordFallbackForAdmins` is true, admins may still use password |
| Membership | User **must already exist** as an active org member |

### No JIT provisioning (current)

Gofox does **not** auto-create users from the first SSO login. Provision the user (invite / admin create) before they sign in with the IdP, or login fails with a not-a-member style error (`SSO_NOT_MEMBER`).

JIT and SCIM are Phase 3 / future work.

---

# Phase 3 — Related & planned

## Social sign-in (not enterprise SSO)

Platform OAuth buttons on the login page (separate from org SSO):

| Method | Path |
|--------|------|
| `GET` | `/api/v1/public/auth/google` + `/callback` |
| `GET` | `/api/v1/public/auth/linkedin` + `/callback` |

Requires `GOOGLE_CLIENT_*` / `LINKEDIN_CLIENT_*` env. Domains under SSO enforcement cannot use these for login.

## Planned

| Feature | Notes |
|---------|--------|
| JIT provisioning | Create membership on first successful SSO |
| SCIM 2.0 | Likely `/api/v1/scim/v2` |
| IdP-initiated SAML | New route + relay state |
| Role mapping from claims | Extend `lib/sso.ts` |

Document new routes here when they ship; update [PHASES.md](./PHASES.md).

---

## Security notes

- Never expose IdP client secrets to the browser
- Validate ACS signatures and audience on every assertion
- Prefer short-lived state / nonce for OIDC
- Rotate IdP certificates before expiry
- Set `API_BASE_URL` and `CLIENT_URL` correctly in each environment

---

## Media placeholders

| File | Purpose |
|------|---------|
| `docs/assets/sso-settings.png` | Tenant SSO settings |
| `docs/assets/sso-login-discover.png` | Login discover / Continue with SSO |
| `docs/assets/sso-okta-setup.mp4` | Okta / Entra walkthrough |

---

## License

Documentation © Gofox. SSO subject to plan entitlements and your IdP agreement.
