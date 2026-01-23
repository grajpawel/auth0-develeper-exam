# Auth0 Developer Exam Study Notes

Comprehensive study materials for the Auth0 Developer Exam, organized by topic.

## 📚 Study Guide Structure

### [01 - Authentication Fundamentals](./01-authentication-fundamentals.md)
- Authentication vs Authorization
- Authentication Factors (Knowledge, Possession, Inherence)
- Authenticators by Factor Type (Knowledge, Possession, Inherence-based)
- Login Credentials
- Auth0 Guardian
- FIDO/FIDO2/WebAuthn
- Authentication Assurance Levels (AAL1, AAL2, AAL3)
- Step-Up Authentication
- Adaptive MFA (Risk-Based Authentication)
- Step-Up vs Adaptive MFA Comparison

### [02 - OAuth 2.0, OpenID Connect, and SAML](./02-oauth-openid-saml.md)
- OAuth 2.0 Framework and Components
- OAuth 2.0 Flows (Authorization Code, PKCE, Client Credentials, etc.)
- Scopes and Scope Validation
- OpenID Connect (OIDC)
- ID Tokens vs Access Tokens
- SAML (Security Assertion Markup Language)
- When to use OAuth 2.0 vs OIDC vs SAML

### [03 - Tokens and Security](./03-tokens-and-security.md)
- Access Tokens (purpose, format, contents, validation)
- ID Tokens (purpose, format, contents, validation)
- Refresh Tokens (rotation, security)
- Client ID and Client Secret
- Confidential vs Public Clients
- Token Storage Best Practices
- Token Security and Validation

### [04 - Auth0 Connections](./04-auth0-connections.md)
- Connection Types Overview
- Database Connections (password policies, custom DB scripts)
- Social Connections (Google, Facebook, etc.)
- Enterprise Connections (SAML, OIDC, AD/LDAP)
- Passwordless Connections (Email, SMS)
- Account Linking Methods
- Connection Selection by Use Case

### [05 - Auth0 Actions and Customization](./05-actions-and-customization.md)
- What are Auth0 Actions
- Action Specifications (Node.js 18, 20s timeout, 5MB size limit)
- Action Triggers (post-login, pre/post-registration, etc.)
- Action API (event and api objects)
- Secrets and Dependencies
- Custom Domains (CCDUL - Configuring Custom Domain for Universal Login)
- Error Pages (when displayed, how to customize)
- Customizing Error Pages (CEP)

### [06 - Multi-Factor Authentication](./06-mfa-and-security.md)
- MFA Fundamentals
- MFA Factors in Auth0 (TOTP, SMS, Email, Push, WebAuthn, etc.)
- Auth0 Guardian (detailed features and configuration)
- MFA Policies (Always, Adaptive, Never)
- Conditional MFA via Actions
- Remember Browser Feature
- MFA Reset and Recovery Codes
- MFA Best Practices

### [07 - Application Types and Use Cases](./07-application-types-use-cases.md)
- Application Types (Regular Web, SPA, Native, M2M)
- When to use each application type
- Public vs Confidential Clients
- B2C (Business-to-Consumer)
- B2B (Business-to-Business)
- B2E (Business-to-Employee)
- Multi-Tenant Scenarios
- Auth0 Organizations

### [08 - RBAC and Authorization](./08-rbac-authorization.md)
- Role-Based Access Control (RBAC)
- RBAC Components (Users, Roles, Permissions, Resources)
- Setting up RBAC in Auth0
- Permissions in Access Tokens
- Scopes vs Permissions
- Authorization Strategies (RBAC, ABAC, ACL, ReBAC)
- API Authorization Implementation
- Authorization Best Practices

## 🎯 Quick Reference Guide

### Key Flows to Memorize

#### Authorization Code Flow with PKCE
1. Client generates `code_verifier` (random string)
2. Creates `code_challenge` = SHA256(code_verifier)
3. Redirects to `/authorize` with code_challenge
4. User authenticates
5. Auth0 returns authorization code
6. Client exchanges code + code_verifier for tokens
7. Auth0 validates: hash(code_verifier) == code_challenge

**Use for**: SPAs, Native/Mobile apps (public clients)

#### Client Credentials Flow
1. Client sends client_id + client_secret to `/oauth/token`
2. Receives access token
3. Uses token to call API

**Use for**: M2M (machine-to-machine), no user involved

### Critical Limits and Timeouts

| Component | Limit | Notes |
|-----------|-------|-------|
| Action Execution Time | 20 seconds | Timeout causes failure |
| Action Code Size | 500 KB | Per action |
| Action Total Size | 5 MB | Including dependencies |
| Action Secrets | 100 | Per action |
| Access Token Default Lifetime | 24 hours | Configurable per API |
| ID Token Default Lifetime | 10 hours | Configurable |
| Remember Browser (MFA) | 30 days | Default, configurable |

### Connection Types Quick Reference

| Connection Type | Use Case | Protocol |
|----------------|----------|----------|
| Database | Own user storage, B2C apps | Username/Password |
| Social | Consumer apps, quick signup | OAuth 2.0 |
| Enterprise | B2B, SSO requirement | SAML, OIDC, LDAP |
| Passwordless | Mobile-first, convenience | Email/SMS codes |

### Application Type Selection

| App Type | Client Type | Flow | Use Client Secret? |
|----------|-------------|------|-------------------|
| Regular Web | Confidential | Authorization Code | ✅ Yes |
| SPA | Public | Auth Code + PKCE | ❌ No |
| Native | Public | Auth Code + PKCE | ❌ No |
| M2M | Confidential | Client Credentials | ✅ Yes |

### Token Comparison

| Token | Purpose | Format | Sent To | Contains |
|-------|---------|--------|---------|----------|
| Access Token | API Authorization | JWT/Opaque | Resource Server (API) | Scopes, permissions, aud, exp |
| ID Token | User Authentication | JWT | Client Application | User claims, sub, iss, aud |
| Refresh Token | Renew Access | Opaque | Authorization Server | N/A (opaque) |

## 🔐 Security Best Practices Checklist

### Tokens
- ✅ Use HTTPS everywhere
- ✅ Short access token lifetimes
- ✅ Enable refresh token rotation
- ✅ Never store tokens in localStorage (XSS risk)
- ✅ Use httpOnly cookies or memory storage
- ✅ Validate signature, iss, aud, exp on all tokens

### Applications
- ✅ Always use PKCE for SPAs and Native apps
- ✅ Never embed client secrets in public clients
- ✅ Whitelist callback URLs (be specific)
- ✅ Use state parameter (CSRF protection)
- ✅ Implement proper CORS policies

### MFA
- ✅ Enable MFA for privileged accounts
- ✅ Prefer hardware keys over SMS
- ✅ Use adaptive MFA for better UX
- ✅ Provide recovery options
- ✅ Monitor MFA reset events

### Authorization
- ✅ Enable RBAC in API settings
- ✅ Add permissions to access tokens
- ✅ Enforce authorization in API (never trust client)
- ✅ Use principle of least privilege
- ✅ Namespace custom claims (https://myapp.com/roles)

## 📝 Common Exam Topics

### Must Know
1. **PKCE**: What it is, why needed, how it works
2. **Client Credentials Flow**: When to use, how it works
3. **Application Types**: Which to use when
4. **Token Types**: Access vs ID vs Refresh
5. **MFA Factors**: Types and security levels
6. **Actions**: Triggers, limits, API methods
7. **RBAC**: Roles, permissions, scopes
8. **Connections**: Database, Social, Enterprise, Passwordless
9. **AAL Levels**: AAL1, AAL2, AAL3 requirements and use cases
10. **Step-Up Authentication**: When and how to implement
11. **Adaptive MFA**: Risk factors and implementation

### Frequently Tested
- Custom domain setup steps
- Error page customization
- Account linking methods
- Action timeout and size limits
- Token validation steps
- B2B vs B2C vs B2E scenarios
- Public vs Confidential clients
- OAuth 2.0 components and flows
- AAL levels and when to require each
- Step-up authentication implementation
- Adaptive MFA risk factors

### Tricky Areas
- **Scopes vs Permissions**: Scopes = app capability, Permissions = user capability
- **OIDC vs OAuth 2.0**: OIDC adds authentication (ID token) to OAuth's authorization
- **Implicit Flow**: DEPRECATED, don't use
- **SMS for MFA**: Less secure than TOTP due to SIM swapping
- **Auto account linking**: Security risk without verification
- **localStorage for tokens**: Never use (XSS vulnerability)
- **Step-Up vs Adaptive MFA**: Step-up = action-based, Adaptive = login-time risk assessment
- **AAL2 vs AAL3**: AAL2 = any MFA, AAL3 = hardware-based MFA required
- **Authenticator factors**: Biometrics = inherence + possession (device required)

## 🎓 Study Tips

1. **Hands-on Practice**: Create applications in Auth0 Dashboard
2. **Test Flows**: Implement each OAuth flow in sample apps
3. **Review Documentation**: Official Auth0 docs are exam source material
4. **Understand Why**: Don't just memorize, understand security reasons
5. **Compare Options**: Know when to use each feature/flow
6. **Security First**: Many questions test security knowledge
7. **Review Limits**: Know timeout, size, and other constraints

## 🔗 Additional Resources

- [Auth0 Documentation](https://auth0.com/docs)
- [Auth0 Learning Center](https://learning.okta.com/page/development)
- [Auth0 Learning Center - Part 2](https://learning.okta.com/page/development-2)
- [OAuth 2.0 RFC 6749](https://tools.ietf.org/html/rfc6749)
- [OpenID Connect Specification](https://openid.net/specs/openid-connect-core-1_0.html)
- [PKCE RFC 7636](https://tools.ietf.org/html/rfc7636)
- [NIST SP 800-63B - Authentication Assurance Levels](https://pages.nist.gov/800-63-3/sp800-63b.html)

## 🧪 Recommended Hands-On Labs

### Lab: Secure Auth0 Applications with MFA (LSAM)
**Focus Areas**:
- Configuring MFA factors in Auth0
- Implementing adaptive MFA using Actions
- Testing different MFA methods (TOTP, SMS, Push, WebAuthn)
- Setting up step-up authentication for sensitive operations
- Understanding AAL levels in practice
- Configuring Guardian for push notifications
- Testing risk-based authentication scenarios

**Key Exercises**:
1. Enable multiple MFA factors for your application
2. Create an Action to enforce AAL2 for admin users
3. Implement step-up authentication for payment operations
4. Build adaptive MFA logic based on IP, location, and device
5. Configure and test Auth0 Guardian
6. Implement AAL3 requirement using WebAuthn/FIDO2
7. Test MFA recovery codes and reset procedures

**Learning Outcomes**:
- Hands-on experience with all MFA factor types
- Understanding when to use AAL1, AAL2, and AAL3
- Implementing conditional/adaptive MFA
- Differentiating step-up vs adaptive authentication
- Securing applications with appropriate assurance levels

## ✅ Pre-Exam Checklist

- [ ] Understand all OAuth 2.0 flows and when to use each
- [ ] Know PKCE in detail (what, why, how)
- [ ] Memorize Action limits (20s timeout, 5MB size)
- [ ] Understand token types and their purposes
- [ ] Know all connection types and use cases
- [ ] Understand RBAC: roles, permissions, scopes
- [ ] Know MFA factors and security levels
- [ ] Understand AAL levels (AAL1, AAL2, AAL3)
- [ ] Know step-up authentication vs adaptive MFA
- [ ] Understand risk factors for adaptive authentication
- [ ] Know authenticator types by factor (knowledge, possession, inherence)
- [ ] Understand application types and client types
- [ ] Know custom domain setup process
- [ ] Understand error page customization
- [ ] Know account linking methods
- [ ] Understand B2B vs B2C vs B2E differences
- [ ] Complete hands-on lab: Secure Auth0 Applications with MFA
- [ ] Test all MFA factors in practice
- [ ] Review security best practices for each component

---

**Good luck with your Auth0 Developer Exam! 🚀**
