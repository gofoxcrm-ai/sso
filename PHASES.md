# SSO — documentation & delivery phases

| Phase | Theme | Status |
|-------|--------|--------|
| **1** | SAML / OIDC setup, ACS, metadata, domain enforcement | ✅ Implemented — [README.md](./README.md#phase-1--enterprise-sso-saml--oidc) |
| **2** | Discover UX, plan entitlement, admin password fallback | ✅ Implemented — [README.md](./README.md#phase-2--login-ux--policy) |
| **3** | Social Google/LinkedIn (separate from enterprise SSO); SCIM / JIT | 🚧 Social live; SCIM/JIT not shipped — [README.md](./README.md#phase-3--related--planned) |

## Phase checklist

### Phase 1
- [x] OIDC authorization code login + callback
- [x] SAML SP-initiated login + ACS
- [x] SP metadata XML
- [x] Encrypted client secret / IdP cert storage
- [x] Plan gate `sso` (Prime+)
- [ ] Screenshots / IdP walkthrough video

### Phase 2
- [x] Email/domain discover endpoint
- [x] Enforce SSO for matched domains (password + social blocked)
- [x] Admin password fallback option
- [x] Document **no JIT** — users must already be org members

### Phase 3
- [x] Google / LinkedIn OAuth login (platform sign-in, not org SSO)
- [ ] JIT user provisioning
- [ ] SCIM 2.0
- [ ] IdP-initiated SAML
- [ ] Fix UI Entity ID copy to match `/metadata` if still drifted

## Source of truth

| Concern | Path |
|---------|------|
| Public SSO routes | `gofox-server/src/modules/auth/public-sso.routes.ts` |
| SSO helpers | `gofox-server/src/lib/sso.ts` |
| Settings | Org settings `ssoSettings` via `/api/v1/tenant/organization/settings` |
| UI | `gofox-client/.../account-settings/ssoSettingsLive.tsx` |
