# OAuth 2.0, OpenID Connect, and SAML

## OAuth 2.0

### What is OAuth 2.0?
- **Authorization** framework (NOT authentication)
- Allows third-party applications to access user resources without sharing credentials
- **Delegated authorization**: "Let app X access my data on service Y"
- Industry standard (RFC 6749)

### OAuth 2.0 Components

#### 1. Resource Owner
- The **user** who owns the data
- Can grant access to their protected resources

#### 2. Resource Server
- Server hosting protected resources (APIs)
- Validates access tokens
- Returns requested resources if token is valid

#### 3. Client
- Application requesting access to resources
- Could be web app, mobile app, SPA, etc.

#### 4. Authorization Server
- Issues access tokens after authenticating user
- In Auth0, this is Auth0 itself
- Handles consent and token issuance

### OAuth 2.0 Grant Types (Flows)

#### Authorization Code Flow
- **Best for**: Server-side web applications
- **Most secure** flow
- Client receives authorization code, exchanges for token
- Tokens never exposed to browser
- **Steps**:
  1. User redirected to authorization server
  2. User authenticates and consents
  3. Authorization code returned to client
  4. Client exchanges code for access token (server-side)
  5. Client uses access token to call API

#### Authorization Code Flow with PKCE
- **Best for**: SPAs, mobile apps, native apps
- **PKCE** = Proof Key for Code Exchange
- Prevents authorization code interception
- **Why needed**: Public clients can't securely store client secret
- **How it works**:
  1. Client generates **code_verifier** (random string)
  2. Creates **code_challenge** (SHA256 hash of verifier)
  3. Sends code_challenge with authorization request
  4. Receives authorization code
  5. Sends authorization code + original code_verifier
  6. Server validates: hash(code_verifier) == code_challenge
  7. Issues access token if valid

**Key Point**: PKCE prevents MITM attacks on authorization code

#### Client Credentials Flow
- **Best for**: Machine-to-machine (M2M) communication
- **No user involved** (server-to-server)
- Client authenticates with client ID + secret
- Directly receives access token
- **Use cases**:
  - Backend service calling API
  - Scheduled jobs
  - CLI tools
- **Steps**:
  1. Client sends client_id + client_secret to token endpoint
  2. Receives access token
  3. Uses token to call API

#### Implicit Flow (DEPRECATED)
- ❌ **No longer recommended**
- Access token returned directly in URL fragment
- Less secure than Authorization Code + PKCE
- Use Authorization Code + PKCE instead

#### Resource Owner Password Credentials (DEPRECATED)
- ❌ **Avoid using**
- User provides username/password directly to client
- Defeats purpose of OAuth
- Only for legacy migration scenarios

### OAuth 2.0 Scopes

#### What are Scopes?
- **Permissions** requested by client
- Define what access token can do
- Set by API, requested by client
- User may consent to scopes

#### Scope Examples
- `read:email` - Read user's email
- `write:posts` - Create posts
- `delete:account` - Delete account
- `openid` - Required for OIDC (returns ID token)
- `profile` - Access user profile info
- `email` - Access user email

#### Where Scopes Are Read From
1. **Client defines** scopes in authorization request
2. **API defines** available scopes in Auth0 Dashboard
3. **Authorization server** validates scopes
4. **Access token** contains granted scopes
5. **Resource server** checks token scopes before allowing access

#### Scope Validation Flow
```
Client Request → Authorization Server validates → 
User consents → Token issued with scopes → 
Resource Server checks scopes in token
```

## OpenID Connect (OIDC)

### What is OIDC?
- **Authentication** layer built on top of OAuth 2.0
- Adds identity verification to OAuth's authorization
- Returns **ID Token** (JWT with user information)
- Industry standard for SSO

### OIDC vs OAuth 2.0
| OAuth 2.0 | OpenID Connect |
|-----------|----------------|
| Authorization | Authentication |
| Access tokens | ID tokens + access tokens |
| "What can you access?" | "Who are you?" |
| Scopes for permissions | Standard claims for identity |

### OIDC Flow
- Same flows as OAuth 2.0, but:
  - Includes `openid` scope in request
  - Returns **ID token** along with access token
  - ID token contains user claims (sub, name, email, etc.)

### ID Token
- **Format**: JWT (JSON Web Token)
- **Contains**: User identity information (claims)
- **Standard Claims**:
  - `sub` - Subject (unique user ID)
  - `name` - Full name
  - `email` - Email address
  - `picture` - Profile picture URL
  - `iss` - Issuer (Auth0 tenant)
  - `aud` - Audience (your client ID)
  - `exp` - Expiration time
  - `iat` - Issued at time

### OIDC Scopes
- `openid` - Required, triggers OIDC
- `profile` - Returns name, picture, etc.
- `email` - Returns email, email_verified
- `address` - Returns address claim
- `phone` - Returns phone_number

### OIDC Request Parameters

#### prompt Parameter
Controls authentication behavior:

| Value | Behavior |
|-------|----------|
| `none` | Silent authentication - no UI shown, fails if auth required |
| `login` | Force re-authentication even if session exists |
| `consent` | Force consent prompt even if previously granted |
| `select_account` | Show account selection even with single account |

**Examples**:
```
# Silent auth - check if session exists
GET /authorize?...&prompt=none

# Force user to re-enter credentials
GET /authorize?...&prompt=login

# Combine values (space-separated)
GET /authorize?...&prompt=login consent
```

**Silent Authentication (prompt=none)**:
- Used to check if user has active session
- Returns tokens silently if session valid
- Fails with `login_required` error if no session
- **Cannot work** with third-party cookies blocked
- Common in SPAs for session check without redirect

#### max_age Parameter
- Forces re-authentication after specified seconds
- `max_age=0` - Always force re-authentication (similar to `prompt=login`)
- `max_age=3600` - Re-authenticate if last auth was > 1 hour ago
- Returns `auth_time` claim in ID token to verify

**Example**:
```
# Re-authenticate if last login was more than 5 minutes ago
GET /authorize?...&max_age=300
```

**max_age vs prompt=login**:
| max_age | prompt=login |
|---------|--------------|
| Re-auth if time exceeded | Always force re-auth |
| Conditional based on time | Unconditional |
| Returns auth_time claim | May not return auth_time |

#### acr_values Parameter
- Request specific Authentication Context Class Reference
- Used for step-up authentication
- Auth0 supports custom ACR values via Actions

**Example**:
```
# Request MFA-based authentication
GET /authorize?...&acr_values=http://schemas.openid.net/pape/policies/2007/06/multi-factor
```

#### login_hint Parameter
- Pre-fill username/email in login form
- Improves UX for known users
- Does NOT skip authentication

**Example**:
```
GET /authorize?...&login_hint=user@example.com
```

#### screen_hint Parameter (Auth0-specific)
- Control which screen to show
- `signup` - Show signup form instead of login

**Example**:
```
GET /authorize?...&screen_hint=signup
```

## SAML (Security Assertion Markup Language)

### What is SAML?
- **XML-based** authentication standard
- Older than OIDC (early 2000s)
- Used for **enterprise SSO**
- Common in B2B, enterprise applications

### SAML Components

#### Service Provider (SP)
- The application user wants to access
- Trusts Identity Provider for authentication
- Your application in Auth0 context

#### Identity Provider (IdP)
- Authenticates users
- Issues SAML assertions
- Auth0 can act as IdP or SP

#### SAML Assertion
- XML document containing:
  - User identity (subject)
  - Authentication method
  - Attributes (email, groups, etc.)
  - Signature for validation

### SAML Flow
1. User attempts to access Service Provider
2. SP generates SAML authentication request
3. User redirected to Identity Provider
4. IdP authenticates user
5. IdP generates signed SAML assertion
6. User redirected back to SP with assertion
7. SP validates assertion signature
8. User granted access

### SAML vs OIDC

| SAML | OIDC |
|------|------|
| XML-based | JSON-based |
| Enterprise legacy | Modern standard |
| More complex | Simpler |
| Desktop/web apps | Web, mobile, SPA |
| SAML assertion | ID token (JWT) |

### When to Use Each

**Use SAML**:
- Enterprise B2B integration
- Legacy systems requirement
- Customer requires SAML
- Desktop applications

**Use OIDC**:
- Modern web/mobile apps
- SPAs, native apps
- Developer-friendly
- Microservices

## Key Exam Takeaways

✅ **OAuth 2.0 = Authorization**, OIDC = Authentication, SAML = Enterprise SSO  
✅ **PKCE required** for public clients (SPAs, mobile apps)  
✅ **Client Credentials** for M2M, no user involved  
✅ **Scopes** defined by API, requested by client, validated by authorization server  
✅ **ID Token** (OIDC) contains user identity, Access Token for API authorization  
✅ **SAML uses XML**, OIDC uses JSON/JWT  
✅ **Authorization Code + PKCE** is most secure for apps with users  
✅ **Implicit Flow deprecated**, always use Authorization Code + PKCE  
✅ **openid scope** required to trigger OIDC and receive ID token  
✅ **OAuth components**: Resource Owner, Client, Authorization Server, Resource Server  
✅ **prompt=none**: Silent authentication, no UI, fails if session required  
✅ **prompt=login**: Force re-authentication even with active session  
✅ **max_age**: Re-authenticate if time since last auth exceeds value  
✅ **login_hint**: Pre-fill username, does NOT skip auth  
✅ **screen_hint=signup**: Show signup form instead of login (Auth0-specific)  
✅ **acr_values**: Request specific authentication level (e.g., MFA)  
