# Authentication & SSO

Nexplane supports two authentication modes per organization: built-in **local** authentication and **OIDC single sign-on** delegated to an external identity provider. The mode is selected per org and can be switched atomically by an admin.

## Local Authentication

Local auth is the default for every organization.

- **JWT + bcrypt** — users log in via `POST /auth/login` and receive a JWT
- All API endpoints require a valid Bearer JWT; tokens expire and require re-authentication
- Passwords are hashed with bcrypt and never stored or logged in plaintext

No external dependencies are required — local auth works out of the box.

## OIDC Single Sign-On

OIDC SSO delegates login to an external identity provider (Okta, Entra ID, Google, Keycloak, or any compliant OIDC provider) via the standard authorization-code flow.

### Login flow

1. The user clicks **Continue with &lt;provider&gt;** on the login screen
2. Nexplane redirects to the provider: `GET /auth/oidc/{idp_id}/redirect` → issuer authorization endpoint
3. The provider authenticates the user and redirects back to `GET /auth/oidc/{idp_id}/callback`
4. Nexplane verifies the `state` parameter, exchanges the code for tokens, and issues a Nexplane JWT

Issuer metadata (authorization, token, and JWKS endpoints) is discovered automatically from the provider's `.well-known/openid-configuration` document, so only the issuer URL, client ID, and client secret need to be configured.

### Identity provider management

Identity providers are managed per organization under `/identity-providers`:

| Endpoint | Purpose |
|----------|---------|
| `POST /identity-providers` | Register a provider (issuer, client ID, secret, scopes) |
| `GET /identity-providers` | List configured providers (admin) |
| `GET /identity-providers/active` | Public — lets the login screen render "Continue with …" buttons before authentication |
| `PUT /identity-providers/{id}` | Update a provider |
| `DELETE /identity-providers/{id}` | Remove a provider |

`GET /identity-providers/active` is intentionally public so the login page can show provider buttons to unauthenticated users. It returns only display-safe fields — never the client secret.

### Per-org auth modes

Each organization runs in one of two auth modes, switched atomically by an admin via `POST /orgs/{org_id}/auth-mode`:

| Mode | Behavior |
|------|----------|
| `local` | Username/password login only |
| `idp` | Login delegated to the active OIDC provider |

- Switching to `idp` activates the chosen provider.
- Switching back to `local` returns providers to `pending`, so a previously tested provider can be re-enabled later without reconfiguration.

### User provisioning

On a successful OIDC callback, Nexplane matches the returned email against existing users scoped to the organization — the match key is `(email, org)`.

- If a user with that email exists in the org, they are logged in.
- If no user exists and the provider has **`auto_provision` enabled**, a new user is created with the `security_operator` role. Auto-provisioned users can log in **only** through the IdP.
- If no user exists and `auto_provision` is **disabled**, login is rejected — this lets admins pre-create accounts and control exactly who may sign in.

!!! note "Redirect URIs"
    OIDC redirect URIs are built from `INSTANCE_URL`. Set this environment variable to the public URL of your Nexplane instance so the callback URL registered with your identity provider matches.

## See also

- [Security Model](model.md) — roles and authorization
- [Safety Engine](safety-engine.md) — how authentication decisions are enforced
