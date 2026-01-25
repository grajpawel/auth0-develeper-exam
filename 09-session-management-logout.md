# Session Management and Logout

## Session Management Fundamentals

### What is Session Management?

Session management controls how long users remain authenticated and how their authentication state is maintained across multiple applications and services.

### Session Layers in Auth0

Auth0 applications typically have **three session layers**:

#### 1. Application Session
- **Where**: Your application (frontend or backend)
- **Managed by**: Your application code
- **Storage**: 
  - Cookies (server-side web apps)
  - Local storage / memory (SPAs)
  - Secure storage (mobile apps)
- **Lifetime**: Controlled by you

#### 2. Auth0 Session (SSO Session)
- **Where**: Auth0 authorization server
- **Managed by**: Auth0
- **Storage**: Encrypted cookie on Auth0 domain
- **Lifetime**: Configurable (default: 3 days inactivity, 7 days absolute)
- **Purpose**: Enable Single Sign-On (SSO) across applications

#### 3. Identity Provider Session
- **Where**: External IdP (Google, Azure AD, SAML provider, etc.)
- **Managed by**: Identity provider
- **Purpose**: User logged into the IdP itself
- **Example**: User logged into Google account

## Single Sign-On (SSO) Sessions

### What is SSO?

Single Sign-On allows users to authenticate once and access multiple applications without re-entering credentials.

### How SSO Works in Auth0

1. User logs into App A via Auth0
2. Auth0 creates SSO session (cookie)
3. User navigates to App B (same Auth0 tenant)
4. App B redirects to Auth0 for authentication
5. Auth0 detects existing SSO session
6. User automatically authenticated to App B (no login prompt)

### SSO Session Configuration

**Dashboard → Tenant Settings → Advanced**

**Inactivity Timeout**:
- Default: 3 days
- Max: 100 days
- Session expires if user inactive
- Reset on each authentication

**Require Login After**:
- Default: 7 days
- Absolute maximum session lifetime
- Requires re-authentication after this period
- Cannot be extended

### SSO Cookie

- **Name**: `auth0` (or custom)
- **Domain**: Auth0 tenant domain
- **HttpOnly**: Yes (security)
- **Secure**: Yes (HTTPS only)
- **SameSite**: None (for cross-domain SSO)

## Logout Options

### Types of Logout

#### 1. Application Logout (Local Logout)

**What it does**:
- Clears application session only
- User logged out of current app
- Auth0 SSO session remains active
- User still logged into other apps

**Implementation**:
```javascript
// Clear local application session
localStorage.removeItem('access_token');
sessionStorage.clear();
// Or destroy server-side session
```

**Use Case**:
- User wants to log out of one app but stay logged into others
- Shared device, but continuing to use other services

#### 2. Auth0 Logout (SSO Logout)

**What it does**:
- Terminates Auth0 SSO session
- Clears application session
- User logged out of all apps in Auth0 tenant
- IdP session may remain active

**Implementation**:
```javascript
// Auth0 SDK
await auth0.logout({
  returnTo: window.location.origin
});

// Direct URL
https://YOUR_DOMAIN/v2/logout?
  client_id=YOUR_CLIENT_ID&
  returnTo=https://yourapp.com
```

**Parameters**:
- `client_id`: Application client ID
- `returnTo`: URL to redirect after logout (must be in Allowed Logout URLs)
- `federated`: Optional, for federated logout

**Use Case**:
- User wants to log out completely
- Security requirement
- Shared/public computer

#### 3. Federated Logout

**What it does**:
- Terminates Auth0 SSO session
- Terminates application session
- **Also terminates IdP session** (Google, SAML provider, etc.)
- Complete logout across all layers

**Implementation**:
```javascript
await auth0.logout({
  returnTo: window.location.origin,
  federated: true  // Key parameter
});

// Direct URL
https://YOUR_DOMAIN/v2/logout?
  client_id=YOUR_CLIENT_ID&
  returnTo=https://yourapp.com&
  federated
```

**When to Use**:
- ✅ Enterprise SSO scenarios (SAML, OIDC)
- ✅ Shared/public computers
- ✅ Compliance requirements
- ✅ High-security applications

**Federated Logout is**:
- ✅ **A best practice to warn the user before ending the third-party IdP session**
- ✅ **Typically used with enterprise IdPs**
- ⚠️ Not always supported by all IdPs
- ⚠️ May impact user experience (logged out of Gmail, etc.)

**Use Case**:
- Corporate applications using Azure AD
- SAML-based enterprise SSO
- Kiosk or shared terminal applications
- Compliance requirements (healthcare, finance)

#### 4. Back-Channel Logout

**What it is**:
- Server-to-server logout notification
- IdP notifies Auth0 of logout
- Auth0 can notify relying parties (applications)
- No browser redirect required

**How It Works**:
1. User logs out of IdP
2. IdP sends back-channel logout request to Auth0
3. Auth0 invalidates SSO session
4. Auth0 optionally notifies applications
5. Applications invalidate their sessions

**Configuration**:
- Supported for OIDC connections
- Configure back-channel logout URI
- Implement logout endpoint in your application

**Application Endpoint**:
```javascript
// POST /backchannel-logout
app.post('/backchannel-logout', (req, res) => {
  const logoutToken = req.body.logout_token;
  
  // Validate logout token
  const decoded = jwt.verify(logoutToken, publicKey);
  
  // Extract session info
  const sid = decoded.sid; // Session ID
  const sub = decoded.sub; // User ID
  
  // Invalidate application session
  invalidateSession(sid);
  
  res.status(200).send('OK');
});
```

**Benefits**:
- ✅ More reliable than front-channel
- ✅ Works without active browser session
- ✅ Better for APIs and background processes
- ✅ No user interaction required

**Use Case**:
- Enterprise SSO with OIDC
- Microservices architecture
- Background job/service sessions
- High-security requirements

#### 5. Global Logout

**What it is**:
- Logout across ALL sessions and devices
- Revokes all tokens for user
- Terminates all active sessions everywhere

**Implementation**:
```javascript
// Using Management API
PATCH /api/v2/users/{user_id}
{
  "blocked": false,
  "app_metadata": {
    "force_logout": Date.now()
  }
}

// Revoke all refresh tokens
DELETE /api/v2/users/{user_id}/multifactor/actions/invalidate-remember-browser
```

**Check in Action**:
```javascript
exports.onExecutePostLogin = async (event, api) => {
  const forceLogout = event.user.app_metadata?.force_logout;
  const lastLogin = event.authentication.initial_login_time;
  
  // If force_logout timestamp is after last login, deny
  if (forceLogout && forceLogout > lastLogin) {
    api.access.deny('Your session has been terminated. Please log in again.');
  }
};
```

**Use Case**:
- User reports compromised account
- Admin terminates user access
- Password reset requiring re-authentication
- Suspicious activity detected

### Logout Comparison Table

| Logout Type | App Session | Auth0 SSO | IdP Session | Other Apps | Use Case |
|-------------|-------------|-----------|-------------|------------|----------|
| **Application** | ✅ Cleared | ❌ Active | ❌ Active | ❌ Logged in | Logout of one app only |
| **Auth0 (SSO)** | ✅ Cleared | ✅ Cleared | ❌ Active | ✅ Logged out | Logout of all Auth0 apps |
| **Federated** | ✅ Cleared | ✅ Cleared | ✅ Cleared | ✅ Logged out | Complete logout everywhere |
| **Back-Channel** | ✅ Cleared | ✅ Cleared | Varies | ✅ Logged out | Server-initiated logout |
| **Global** | ✅ All devices | ✅ All devices | Varies | ✅ All devices | Revoke all access |

## Understanding Application Session Management with Auth0

### Session Management Strategies

#### Strategy 1: Short-Lived Tokens (Stateless)
```javascript
// Access token expires in 1 hour
// No refresh token
// User must re-authenticate hourly
// Most secure, worst UX
```

**Pros**: Maximum security, no session storage  
**Cons**: Frequent re-authentication required

#### Strategy 2: Refresh Tokens (Semi-Stateless)
```javascript
// Access token: 1 hour
// Refresh token: 30 days
// Silent token renewal via refresh token
// Good security + UX balance
```

**Pros**: Long sessions, tokens can be revoked  
**Cons**: More complex, need secure refresh token storage

#### Strategy 3: Silent Authentication (SSO)
```javascript
// Access token expires
// Use Auth0 SSO session to get new token
// iframe-based silent auth
// No user interaction
```

**Pros**: Seamless UX, leverages SSO  
**Cons**: Third-party cookie restrictions, complexity

### Session Timeout Scenarios

#### Scenario 1: User Inactive
```
User last action: 10:00 AM
Inactivity timeout: 30 minutes
Current time: 10:35 AM
Result: Session expired, require re-authentication
```

#### Scenario 2: Absolute Timeout
```
User login: Monday 9:00 AM
Absolute timeout: 7 days
Current time: Monday (next week) 10:00 AM
Result: Session expired, require re-authentication
(Even if user was active)
```

#### Scenario 3: Token Expiration
```
Access token issued: 10:00 AM
Access token lifetime: 1 hour
Current time: 11:05 AM
Refresh token: Available
Result: Silently refresh access token, session continues
```

### Best Practices

#### Logout
✅ **Always provide logout option** in UI  
✅ **Clear all local tokens/session data** on logout  
✅ **Redirect to Auth0 logout endpoint** for SSO logout  
✅ **Use federated logout** for enterprise/shared devices  
✅ **Warn users** before federated logout (impacts IdP session)  
✅ **Configure Allowed Logout URLs** in Auth0 application settings  
✅ **Log logout events** for audit trail  

#### Session Management
✅ **Set appropriate SSO session timeouts** based on security needs  
✅ **Implement refresh token rotation** for long-lived sessions  
✅ **Use absolute maximum lifetime** to force periodic re-auth  
✅ **Monitor active sessions** (enterprise feature)  
✅ **Provide session visibility** to users (where I'm logged in)  
✅ **Implement forced logout** capability for security incidents  

#### Security
✅ **Use HttpOnly cookies** for session storage (web apps)  
✅ **Implement CSRF protection** for logout endpoints  
✅ **Validate returnTo URLs** against allowlist  
✅ **Handle logout failures** gracefully  
✅ **Clear sensitive data** from browser on logout  
✅ **Test logout** across all session layers  

## Multi-Tenancy vs Single Tenancy

### Single Tenancy

**What it is**:
- One Auth0 tenant per customer/organization
- Complete isolation between customers
- Dedicated infrastructure per tenant

**Architecture**:
```
Customer A → Auth0 Tenant A
Customer B → Auth0 Tenant B
Customer C → Auth0 Tenant C
```

**Characteristics**:
- ✅ **Complete isolation** (data, config, customization)
- ✅ **Independent branding** per customer
- ✅ **Custom domains** per tenant
- ✅ **Separate compliance** requirements
- ❌ **Higher cost** (more tenants)
- ❌ **More complex management** (multiple tenants)
- ❌ **No cross-tenant SSO**

**When to Use**:
- **Reseller/White-Label** scenarios
- **Regulatory compliance** requires data isolation
- **Large enterprise customers** needing dedicated tenant
- **Different geographic regions** (data residency)
- **Completely different applications** per customer

**Example Use Cases**:
- SaaS platform selling white-labeled solution
- MSP providing identity services to customers
- Separate tenants for dev/staging/production
- Geographic data residency requirements

### Multi-Tenancy

**What it is**:
- Single Auth0 tenant shared across multiple customers/organizations
- Logical isolation using Auth0 Organizations or metadata
- Shared infrastructure

**Architecture**:
```
Customer A ┐
Customer B ├→ Auth0 Tenant (single)
Customer C ┘
```

**Characteristics**:
- ✅ **Cost-effective** (single tenant)
- ✅ **Easier management** (one tenant to configure)
- ✅ **Shared infrastructure** (economies of scale)
- ✅ **Cross-organization features** possible
- ❌ **Shared compliance scope**
- ❌ **Less isolation** (logical vs physical)
- ⚠️ **Requires careful user/data segregation**

**When to Use**:
- **B2B SaaS** with many small-medium business customers
- **Standard product** for all customers
- **Shared features/integrations** across customers
- **Cost optimization** priority
- **Similar compliance** requirements across customers

**Example Use Cases**:
- B2B project management tool
- CRM with multiple company customers
- Analytics platform with organization workspaces
- Collaboration tools with team separation

### Implementing Multi-Tenancy

#### Method 1: Auth0 Organizations (Recommended)

**What are Organizations**:
- Native Auth0 feature for multi-tenancy
- Logical separation within single tenant
- Organization-specific connections, branding, metadata

**Features**:
- Organization ID in tokens
- Organization-specific SSO
- Member management per organization
- Organization metadata
- Organization-level branding

**Implementation**:
```javascript
// Login with organization
await auth0.loginWithRedirect({
  organization: 'org_abc123'  // Organization ID
});

// Token contains organization
{
  "sub": "auth0|123456",
  "org_id": "org_abc123",
  "org_name": "Acme Corporation"
}

// Scope data by organization
const data = await fetchData({
  organizationId: token.org_id
});
```

**Benefits**:
- ✅ Built-in organization management
- ✅ Organization-specific connections (per-org SAML)
- ✅ Cleaner architecture
- ✅ Native support in Auth0 SDKs

#### Method 2: User Metadata

**Implementation**:
```javascript
// Store organization in user metadata
{
  "user_id": "auth0|123456",
  "app_metadata": {
    "organization_id": "org_abc",
    "tenant_id": "acme_corp"
  }
}

// Add to token via Action
exports.onExecutePostLogin = async (event, api) => {
  const orgId = event.user.app_metadata.organization_id;
  api.accessToken.setCustomClaim('https://myapp.com/org_id', orgId);
};

// Application filters by organization
app.get('/api/data', (req, res) => {
  const orgId = req.user['https://myapp.com/org_id'];
  const data = db.query('SELECT * FROM data WHERE org_id = ?', [orgId]);
  res.json(data);
});
```

**Benefits**:
- ✅ Simple implementation
- ✅ Flexible metadata structure
- ✅ Works with any Auth0 plan

**Drawbacks**:
- ❌ Manual management
- ❌ No native organization features
- ❌ Requires careful implementation

#### Method 3: Separate Connections

**Implementation**:
- Create separate connection per organization
- Database connection per customer
- SAML/OIDC connection per enterprise customer

**Benefits**:
- ✅ Connection-level isolation
- ✅ Per-org authentication settings

**Drawbacks**:
- ❌ Connection limit per tenant
- ❌ Complex management at scale

### Multi-Tenancy Best Practices

#### Data Isolation
✅ **Always filter queries** by organization/tenant ID  
✅ **Validate user belongs** to requested organization  
✅ **Never trust client-provided** tenant ID  
✅ **Use organization ID from token**  
✅ **Test cross-tenant access** prevention  
✅ **Audit data access** by organization  

#### Security
✅ **Validate organization claim** in tokens  
✅ **Implement organization switching** with re-authentication  
✅ **Log organization context** in all audit logs  
✅ **Separate encryption keys** per organization (if needed)  
✅ **Test tenant isolation** thoroughly  

#### User Management
✅ **Use Organizations feature** when possible  
✅ **Support org admins** (delegated administration)  
✅ **Allow users in multiple orgs** (if needed)  
✅ **Implement org invitation** workflow  
✅ **Handle org membership** changes  

### Single Tenancy vs Multi-Tenancy Decision Matrix

| Factor | Single Tenancy | Multi-Tenancy |
|--------|----------------|---------------|
| **Cost** | Higher (multiple tenants) | Lower (one tenant) |
| **Isolation** | Complete (physical) | Logical only |
| **Management** | Complex (many tenants) | Simpler (one tenant) |
| **Customization** | Per-tenant branding | Shared or org-level |
| **Compliance** | Tenant-level | Shared scope |
| **Data Residency** | Per-tenant control | Shared location |
| **SSO** | Per tenant only | Cross-organization possible |
| **Scaling** | Horizontal (more tenants) | Vertical (one tenant) |
| **Best For** | Large enterprises, resellers | B2B SaaS, many small orgs |

## Key Exam Takeaways

✅ **Three session layers**: Application, Auth0 SSO, Identity Provider  
✅ **Application logout**: Clears app session only, SSO remains  
✅ **Auth0 logout**: Clears app + SSO session, IdP remains  
✅ **Federated logout**: Clears all three layers (app + SSO + IdP)  
✅ **Federated logout best practice**: Warn user before ending IdP session  
✅ **Federated logout typically used**: With enterprise IdPs (SAML, OIDC)  
✅ **Back-channel logout**: Server-to-server, no browser redirect  
✅ **Global logout**: Revoke all tokens/sessions across all devices  
✅ **SSO session default**: 3 days inactivity, 7 days absolute  
✅ **returnTo parameter**: Must be in Allowed Logout URLs  
✅ **Single tenancy**: One Auth0 tenant per customer, complete isolation  
✅ **Multi-tenancy**: Single tenant, multiple organizations, logical isolation  
✅ **Auth0 Organizations**: Native multi-tenancy feature, recommended approach  
✅ **Organization ID in token**: Use to scope data access per tenant  
✅ **Multi-tenant security**: Always filter by org ID, validate token claims  
