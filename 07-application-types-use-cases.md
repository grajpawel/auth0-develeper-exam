# Application Types and Use Cases

## Auth0 Application Types

When creating an application in Auth0, you select a type that determines authentication flow and security characteristics.

### Application Type Categories

#### 1. Regular Web Applications

**Description**:
- Traditional server-side web applications
- Code runs on web server (backend)
- Can securely store secrets
- Examples: ASP.NET, Java Spring, Node.js Express, Ruby on Rails, PHP Laravel

**Authentication Flow**:
- **Authorization Code Flow**
- Server exchanges code for tokens
- Tokens stored server-side
- Session cookie to browser

**Characteristics**:
- ✅ **Confidential client** (has client secret)
- ✅ Can use client secret for authentication
- ✅ Tokens never exposed to browser
- ✅ Most secure for web apps

**Use Cases**:
- Traditional web applications with backend
- Server-rendered pages (SSR)
- Applications needing server-side API calls

**Configuration**:
- Client ID + Client Secret
- Allowed callback URLs (server endpoints)
- Server-side token validation

#### 2. Single Page Applications (SPAs)

**Description**:
- JavaScript applications running in browser
- React, Angular, Vue.js, etc.
- No traditional backend or minimal API
- All code runs client-side

**Authentication Flow**:
- **Authorization Code Flow with PKCE**
- No client secret (public client)
- Tokens handled in browser
- PKCE prevents code interception

**Characteristics**:
- ❌ **Public client** (cannot store secrets)
- ✅ Uses PKCE for security
- ⚠️ Tokens accessible to JavaScript
- ⚠️ Vulnerable to XSS if not careful

**Use Cases**:
- Modern JavaScript frameworks
- Progressive Web Apps (PWAs)
- Frontend-heavy applications
- Jamstack applications

**Configuration**:
- Client ID only (no secret)
- Enable PKCE
- Allowed callback URLs (frontend routes)
- Token storage strategy (memory, sessionStorage)

**Security Best Practices**:
- ✅ Always use PKCE
- ✅ Store tokens in memory when possible
- ✅ Use httpOnly cookies if backend available
- ✅ Implement proper CORS
- ✅ Avoid localStorage for tokens
- ✅ Use short token lifetimes

#### 3. Native Applications

**Description**:
- Mobile apps (iOS, Android)
- Desktop applications
- Installed applications
- Examples: Mobile apps, Electron apps, .NET desktop apps

**Authentication Flow**:
- **Authorization Code Flow with PKCE**
- Uses system browser for login (security best practice)
- Deep linking to return to app
- Platform-specific secure storage

**Characteristics**:
- ❌ **Public client** (can be decompiled)
- ✅ Uses PKCE
- ✅ Secure storage available (Keychain, Keystore)
- ✅ Biometric protection possible

**Use Cases**:
- iOS/Android mobile applications
- Desktop applications
- Smart TV apps
- Gaming consoles

**Configuration**:
- Client ID only
- Enable PKCE
- Custom URL scheme for callbacks (e.g., `myapp://callback`)
- Configure allowed callback URLs

**Security Best Practices**:
- ✅ Use system browser (not embedded WebView)
- ✅ Store tokens in secure storage:
  - iOS: Keychain
  - Android: Keystore/EncryptedSharedPreferences
- ✅ Use biometric lock for sensitive apps
- ✅ Implement certificate pinning
- ✅ Use refresh token rotation

#### 4. Machine-to-Machine (M2M) Applications

**Description**:
- Server-to-server communication
- Backend services
- No user interaction
- Automated processes

**Authentication Flow**:
- **Client Credentials Flow**
- Direct token request with client ID + secret
- No user involved
- Service account authentication

**Characteristics**:
- ✅ **Confidential client**
- ✅ Client secret required
- ✅ No user context (service identity)
- ✅ Scopes-based permissions

**Use Cases**:
- Microservices communication
- Backend job/cron processes
- CLI tools (server-side)
- IoT devices (server-side)
- System integration
- Data synchronization

**Configuration**:
- Client ID + Client Secret
- Authorized APIs and scopes
- Token endpoint authentication

**Security Best Practices**:
- ✅ Rotate client secrets regularly
- ✅ Use principle of least privilege (minimal scopes)
- ✅ Store secrets securely (environment variables, secret managers)
- ✅ Monitor M2M token usage
- ✅ One M2M app per service/purpose

## Business Context: B2B, B2C, B2E

### B2C (Business-to-Consumer)

**Definition**: Applications for individual consumers

**Characteristics**:
- Large user base (thousands to millions)
- Self-service registration
- Social login important
- Password reset flows critical
- Email verification
- Consumer-friendly UX

**Auth0 Features**:
- Database connections (own user storage)
- Social connections (Google, Facebook, Apple, etc.)
- Passwordless authentication
- Simple MFA (SMS, TOTP)
- Universal Login for consistent UX
- Account linking for multiple identities

**Examples**:
- E-commerce websites
- Mobile gaming apps
- Streaming services
- Social media platforms
- Consumer SaaS tools

**Key Considerations**:
- ✅ Low friction signup
- ✅ Progressive profiling
- ✅ Social login options
- ✅ Mobile-first experience
- ✅ Scalability for millions of users
- ✅ Passwordless for convenience

### B2B (Business-to-Business)

**Definition**: Applications used by business customers/organizations

**Characteristics**:
- Multiple organizations (tenants)
- SSO integration required
- Organization-level settings
- Admin delegation to customers
- User provisioning (SCIM)
- Compliance requirements

**Auth0 Features**:
- Enterprise connections (SAML, OIDC)
- Organizations (multi-tenancy)
- Home Realm Discovery (route by email domain)
- JIT (Just-In-Time) provisioning
- SCIM for user sync
- Role-based access control (RBAC)

**Examples**:
- B2B SaaS platforms
- Project management tools
- CRM systems
- Analytics platforms
- Collaboration tools

**Key Considerations**:
- ✅ SSO integration (customer's IdP)
- ✅ Multi-tenant architecture
- ✅ Admin delegation per organization
- ✅ Custom domain often required
- ✅ SAML support essential
- ✅ User provisioning automation
- ✅ Compliance (SOC2, GDPR, HIPAA)

### B2E (Business-to-Employee)

**Definition**: Internal applications for company employees

**Characteristics**:
- Single organization
- Corporate identity provider
- Strong security requirements
- Centralized IT management
- Active Directory integration common

**Auth0 Features**:
- Active Directory / LDAP integration
- SAML / OIDC with corporate IdP
- Mandatory MFA
- Conditional access policies
- Integration with HR systems
- Privileged access management

**Examples**:
- Internal portals
- HR systems
- Employee dashboards
- IT management tools
- Finance systems

**Key Considerations**:
- ✅ Integration with corporate IdP (Azure AD, Okta, etc.)
- ✅ Strong MFA requirements
- ✅ Compliance and audit logs
- ✅ On-premises AD integration (connector)
- ✅ Windows Integrated Auth
- ✅ Role-based access (department, job title)

## When to Use Each Application Type

### Regular Web Application
- ✅ Traditional web app with server-side rendering
- ✅ Need to securely store and use client secret
- ✅ API calls made server-side
- ✅ Maximum security required

### Single Page Application
- ✅ React, Angular, Vue.js, etc.
- ✅ Frontend-only or API backend separate
- ✅ Modern JavaScript framework
- ✅ Cannot use client secret

### Native Application
- ✅ Mobile app (iOS/Android)
- ✅ Desktop application
- ✅ Need biometric authentication
- ✅ Want to use secure device storage

### Machine-to-Machine
- ✅ Backend service calling API
- ✅ No user involved
- ✅ Scheduled jobs
- ✅ Microservices authentication
- ✅ System integration

## Multi-Tenant Scenarios

### What is Multi-Tenancy?

Multiple organizations/customers using same application instance:
- Each organization is a "tenant"
- Isolated data and settings per tenant
- Shared infrastructure

### Auth0 Organizations Feature

**Purpose**: Native multi-tenancy support in Auth0

**Features**:
- Dedicated organization namespace
- Organization-specific connections
- Organization-level branding
- Organization metadata
- Organization invitations
- Member management per organization

**Use Cases**:
- B2B SaaS with many business customers
- Each customer needs own SSO
- Customer-specific branding
- Isolated user management per customer

**Implementation**:
```javascript
// Login with organization
await auth0.loginWithRedirect({
  organization: 'org_abc123'
});

// Organization ID in token
// Use to scope data access per tenant
```

### Creating and Managing Organizations

#### Creating an Organization

**Via Dashboard**:
1. Navigate to **Organizations** in Auth0 Dashboard
2. Click **Create Organization**
3. Provide:
   - **Name**: Display name (e.g., "Acme Corporation")
   - **Organization ID**: Unique identifier (e.g., `org_abc123`)
   - **Display Name**: User-facing name
   - **Logo**: Optional organization logo
   - **Metadata**: Custom key-value pairs

**Via Management API**:
```javascript
POST /api/v2/organizations
{
  "name": "acme-corp",
  "display_name": "Acme Corporation",
  "branding": {
    "logo_url": "https://acme.com/logo.png",
    "colors": {
      "primary": "#0066CC",
      "page_background": "#FFFFFF"
    }
  },
  "metadata": {
    "plan": "enterprise",
    "industry": "technology"
  }
}
```

#### Organization Settings

**Connections**:
- Assign specific connections to organization
- Organization members can only use assigned connections
- Example: Acme Corp uses their Azure AD SAML connection

**Branding**:
- Organization-specific logo
- Custom colors
- Branded login experience per organization

**Metadata**:
- Store organization-specific data
- Plan type, settings, custom attributes
- Accessible in tokens and Actions

### Organization Roles

#### What are Organization Roles?

**Organization Roles** are distinct from regular Auth0 roles:
- **Scoped to organization**: Role only applies within specific organization
- **Separate from global roles**: User can have different roles in different organizations
- **Use case**: User is Admin in Org A, but Member in Org B

#### Organization Roles vs Regular Roles

| Feature | Regular Roles | Organization Roles |
|---------|--------------|-------------------|
| **Scope** | Global (tenant-wide) | Per-organization |
| **Use Case** | Single-tenant apps | Multi-tenant B2B apps |
| **Assignment** | User has role globally | User has role in specific org |
| **Permissions** | Global API permissions | Org-scoped permissions |
| **Token Claim** | `permissions` array | `org_id` + permissions |

#### Creating Organization Roles

**Via Dashboard**:
1. Navigate to **Organizations**
2. Select an organization
3. Go to **Roles** tab
4. Click **Assign Roles** or **Create Role**
5. Define role name and permissions

**Via Management API**:
```javascript
// Create an organization role
POST /api/v2/organizations/{org_id}/roles
{
  "name": "org-admin",
  "description": "Administrator for this organization"
}

// Assign permissions to organization role
POST /api/v2/organizations/{org_id}/roles/{role_id}/permissions
{
  "permissions": [
    {
      "permission_name": "read:organization_members",
      "resource_server_identifier": "https://api.myapp.com"
    },
    {
      "permission_name": "update:organization_settings",
      "resource_server_identifier": "https://api.myapp.com"
    }
  ]
}
```

### Assigning Roles to Organization Members

#### Adding Members to Organization

**Via Dashboard**:
1. Navigate to **Organizations** → Select Organization
2. Go to **Members** tab
3. Click **Add Members**
4. Search and select users
5. Optionally assign roles during addition

**Via Management API**:
```javascript
// Add member to organization
POST /api/v2/organizations/{org_id}/members
{
  "members": [
    "auth0|user123",
    "auth0|user456"
  ]
}

// Add member with specific roles
POST /api/v2/organizations/{org_id}/members
{
  "members": ["auth0|user123"],
  "roles": ["rol_orgadmin123"]
}
```

#### Assigning Roles to Existing Members

**Via Dashboard**:
1. Navigate to **Organizations** → Select Organization
2. Go to **Members** tab
3. Select user
4. Click **Assign Roles**
5. Choose one or more roles
6. Click **Assign**

**Via Management API**:
```javascript
// Assign roles to organization member
POST /api/v2/organizations/{org_id}/members/{user_id}/roles
{
  "roles": [
    "rol_orgadmin123",
    "rol_orgmanager456"
  ]
}

// Remove roles from organization member
DELETE /api/v2/organizations/{org_id}/members/{user_id}/roles
{
  "roles": ["rol_orgadmin123"]
}
```

#### Listing Member Roles

**Via Management API**:
```javascript
// Get roles for a specific member in organization
GET /api/v2/organizations/{org_id}/members/{user_id}/roles

// Response
{
  "roles": [
    {
      "id": "rol_abc123",
      "name": "org-admin",
      "description": "Organization Administrator"
    }
  ]
}

// Get all members with a specific role
GET /api/v2/organizations/{org_id}/roles/{role_id}/users
```

### Organization Invitations

#### What are Organization Invitations?

Invite users to join an organization:
- Send invitation to email address
- User doesn't need to exist in Auth0 yet
- Can assign roles during invitation
- Invitation expires after set period

#### Creating Invitations

**Via Dashboard**:
1. Navigate to **Organizations** → Select Organization
2. Go to **Invitations** tab
3. Click **Invite Members**
4. Enter email address(es)
5. Optionally assign roles
6. Send invitation

**Via Management API**:
```javascript
// Create organization invitation
POST /api/v2/organizations/{org_id}/invitations
{
  "inviter": {
    "name": "Admin User"
  },
  "invitee": {
    "email": "newuser@example.com"
  },
  "client_id": "your_app_client_id",
  "roles": ["rol_orgmember123"],
  "ttl_sec": 604800  // 7 days
}

// Response includes invitation URL
{
  "id": "inv_abc123",
  "ticket_id": "ticket_xyz789",
  "invitation_url": "https://your-tenant.auth0.com/...",
  "expires_at": "2026-02-01T12:00:00.000Z"
}
```

#### Invitation Flow

1. **Admin sends invitation**
   - Creates invitation with email + roles
   - Invitation URL generated

2. **User receives email**
   - Email contains invitation link
   - Link includes organization context

3. **User clicks link**
   - Redirected to Auth0 login/signup
   - Organization context maintained

4. **User authenticates**
   - New user: Creates account
   - Existing user: Signs in

5. **User added to organization**
   - Automatically becomes member
   - Assigned roles applied
   - Can access organization resources

### Organization Permissions in Tokens

#### Organization ID in Tokens

When user logs in with organization:
```javascript
// ID Token
{
  "sub": "auth0|123456",
  "email": "user@example.com",
  "org_id": "org_abc123",
  "org_name": "Acme Corporation"
}

// Access Token
{
  "sub": "auth0|123456",
  "org_id": "org_abc123",
  "permissions": [
    "read:organization_members",
    "update:organization_settings"
  ]
}
```

#### Enforcing Organization Context

**In Actions**:
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Ensure user is member of requested organization
  const requestedOrg = event.organization?.id;
  const userOrgs = event.user.org_member || [];
  
  if (requestedOrg && !userOrgs.includes(requestedOrg)) {
    api.access.deny('You are not a member of this organization');
  }
  
  // Add organization-specific claims
  if (event.organization) {
    api.idToken.setCustomClaim('https://myapp.com/org_role', 
      event.authorization?.roles || []
    );
  }
};
```

**In Your Application**:
```javascript
// Validate organization from token
function validateOrganization(token, expectedOrgId) {
  const decoded = jwt.verify(token, publicKey);
  
  if (decoded.org_id !== expectedOrgId) {
    throw new Error('Invalid organization');
  }
  
  return decoded;
}

// Filter data by organization
app.get('/api/data', (req, res) => {
  const token = validateToken(req);
  const orgId = token.org_id;
  
  // Only return data for user's organization
  const data = db.query(
    'SELECT * FROM data WHERE org_id = ?',
    [orgId]
  );
  
  res.json(data);
});
```

### Organization Member Management Best Practices

#### Role Assignment

✅ **Use meaningful role names**
- "Organization Admin", "Organization Member"
- Not "Role1", "Role2"

✅ **Assign minimum necessary permissions**
- Principle of least privilege
- Grant only what's needed for job function

✅ **Use roles, not individual permissions**
- Easier to manage
- Consistent across organization

✅ **Document role purposes**
- Clear description of what each role does
- Who should have which role

#### Member Lifecycle

✅ **Onboarding**
- Use invitations for new members
- Assign default role during invitation
- Automated welcome workflow

✅ **Role Changes**
- Log all role assignments/removals
- Require approval for privileged roles
- Audit trail for compliance

✅ **Offboarding**
- Remove from organization (not delete user)
- User can still exist in other organizations
- Revoke all organization-specific access

#### Security

✅ **Validate organization membership**
- Always check `org_id` in tokens
- Verify user belongs to organization
- Don't trust client-provided org ID

✅ **Organization isolation**
- Data segregation per organization
- No cross-organization data leakage
- Test isolation thoroughly

✅ **Admin delegation**
- Allow organization admins to manage their members
- Limit to organization scope (not tenant-wide)
- Audit admin actions

### Use Case Examples

#### Example 1: B2B SaaS with Multiple Customers

**Scenario**: Project management tool with multiple company customers

**Setup**:
```javascript
// Organizations
- org_acme: Acme Corporation
- org_globex: Globex Corporation
- org_initech: Initech

// Roles per organization
- org-owner: Full control
- org-admin: Manage members, settings
- org-member: Access projects, create tasks
- org-guest: View-only access

// Member assignments
Acme Corporation:
  - alice@acme.com: org-owner
  - bob@acme.com: org-admin
  - charlie@acme.com: org-member

Globex Corporation:
  - dave@globex.com: org-owner
  - eve@globex.com: org-member
```

**Implementation**:
- Each company is an organization
- Company admin manages their members
- Users can only see their organization's data
- Organization ID used to scope all queries

#### Example 2: User in Multiple Organizations

**Scenario**: Consultant works with multiple clients

**Setup**:
```javascript
User: consultant@example.com

Organization Memberships:
- Acme Corp (org_acme): org-admin
- Globex Corp (org_globex): org-member
- Initech (org_initech): org-guest

// User has different roles in each organization
```

**Login Flow**:
```javascript
// User selects organization at login
await auth0.loginWithRedirect({
  organization: 'org_acme'  // Logging into Acme
});

// Token includes selected organization
{
  "org_id": "org_acme",
  "permissions": ["manage:members", "update:settings"]  // Admin permissions
}

// If logs into Globex instead
{
  "org_id": "org_globex",
  "permissions": ["read:projects", "create:tasks"]  // Member permissions
}
```

#### Example 3: Organization-Specific SSO

**Scenario**: Each customer uses their own identity provider

**Setup**:
```javascript
// Acme uses Azure AD SAML
Organization: org_acme
Connection: acme-azure-ad-saml
Domain: @acme.com

// Globex uses Okta OIDC
Organization: org_globex
Connection: globex-okta-oidc
Domain: @globex.com

// Home Realm Discovery
if (email.endsWith('@acme.com')) {
  // Route to Acme's Azure AD
  organization = 'org_acme'
} else if (email.endsWith('@globex.com')) {
  // Route to Globex's Okta
  organization = 'org_globex'
}
```

### Multi-Tenant Strategies

#### 1. Separate Connections per Tenant
- Create SAML/OIDC connection for each customer
- Home Realm Discovery by email domain
- Each tenant authenticates via their IdP

#### 2. Auth0 Organizations (Recommended)
- Use Organizations feature
- Cleaner architecture
- Better UX
- Easier management

#### 3. Shared Connection with Metadata
- Single database connection
- Tenant ID in user metadata
- Application handles tenant isolation
- Simpler for small number of tenants

## Auth0 SDKs by Application Type

### SDK Selection Guide

| Application Type | Recommended SDK | Alternative |
|-----------------|-----------------|-------------|
| **React SPA** | @auth0/auth0-react | auth0-spa-js |
| **Angular SPA** | @auth0/auth0-angular | auth0-spa-js |
| **Vue.js SPA** | auth0-spa-js + Vue plugin | - |
| **Generic SPA** | @auth0/auth0-spa-js | - |
| **Node.js Web App** | express-openid-connect | passport-auth0 |
| **Python Web App** | authlib | python-social-auth |
| **iOS Native** | Auth0.swift | - |
| **Android Native** | Auth0.Android | - |
| **React Native** | react-native-auth0 | - |
| **Flutter** | auth0_flutter | - |

### SPA SDK: auth0-spa-js

**Best for**: Single Page Applications

**Key Features**:
- ✅ PKCE built-in (automatic)
- ✅ Token caching and renewal
- ✅ Silent authentication
- ✅ TypeScript support
- ✅ Small bundle size

**Installation**:
```bash
npm install @auth0/auth0-spa-js
```

**Basic Usage**:
```javascript
import { createAuth0Client } from '@auth0/auth0-spa-js';

const auth0 = await createAuth0Client({
  domain: 'your-tenant.auth0.com',
  clientId: 'your-client-id',
  authorizationParams: {
    redirect_uri: window.location.origin,
    audience: 'https://api.myapp.com'
  }
});

// Login
await auth0.loginWithRedirect();

// Handle callback
await auth0.handleRedirectCallback();

// Get access token
const token = await auth0.getTokenSilently();

// Get user info
const user = await auth0.getUser();

// Logout
await auth0.logout({ 
  logoutParams: { returnTo: window.location.origin }
});
```

### React SDK: auth0-react

**Best for**: React applications

**Key Features**:
- React hooks (useAuth0)
- Context provider
- Built on auth0-spa-js

**Installation**:
```bash
npm install @auth0/auth0-react
```

**Basic Usage**:
```jsx
// App.js - Provider setup
import { Auth0Provider } from '@auth0/auth0-react';

<Auth0Provider
  domain="your-tenant.auth0.com"
  clientId="your-client-id"
  authorizationParams={{
    redirect_uri: window.location.origin,
    audience: 'https://api.myapp.com'
  }}
>
  <App />
</Auth0Provider>

// Component usage
import { useAuth0 } from '@auth0/auth0-react';

function Profile() {
  const { 
    isAuthenticated, 
    isLoading, 
    user, 
    loginWithRedirect, 
    logout,
    getAccessTokenSilently 
  } = useAuth0();

  if (isLoading) return <div>Loading...</div>;
  
  if (!isAuthenticated) {
    return <button onClick={() => loginWithRedirect()}>Login</button>;
  }

  return (
    <div>
      <img src={user.picture} alt={user.name} />
      <h2>{user.name}</h2>
      <button onClick={() => logout()}>Logout</button>
    </div>
  );
}
```

### Mobile SDKs

#### iOS (Swift)

```swift
import Auth0

// Login
Auth0
    .webAuth()
    .audience("https://api.myapp.com")
    .scope("openid profile email")
    .start { result in
        switch result {
        case .success(let credentials):
            print("Access Token: \(credentials.accessToken)")
        case .failure(let error):
            print("Error: \(error)")
        }
    }

// Logout
Auth0
    .webAuth()
    .clearSession { result in
        // Handle logout
    }
```

#### Android (Kotlin)

```kotlin
import com.auth0.android.Auth0
import com.auth0.android.authentication.AuthenticationException
import com.auth0.android.provider.WebAuthProvider
import com.auth0.android.callback.Callback
import com.auth0.android.result.Credentials

val auth0 = Auth0(clientId, domain)

// Login
WebAuthProvider.login(auth0)
    .withScheme("demo")
    .withAudience("https://api.myapp.com")
    .start(this, object : Callback<Credentials, AuthenticationException> {
        override fun onSuccess(credentials: Credentials) {
            val accessToken = credentials.accessToken
        }
        override fun onFailure(error: AuthenticationException) {
            // Handle error
        }
    })

// Logout
WebAuthProvider.logout(auth0)
    .withScheme("demo")
    .start(this, callback)
```

### Server-Side SDK: express-openid-connect

**Best for**: Node.js Express applications

**Key Features**:
- Session management built-in
- Server-side token handling
- Middleware-based

**Installation**:
```bash
npm install express-openid-connect
```

**Basic Usage**:
```javascript
const { auth, requiresAuth } = require('express-openid-connect');

app.use(
  auth({
    authRequired: false,
    auth0Logout: true,
    secret: process.env.SESSION_SECRET,
    baseURL: 'https://myapp.com',
    clientID: 'your-client-id',
    issuerBaseURL: 'https://your-tenant.auth0.com'
  })
);

// Protected route
app.get('/profile', requiresAuth(), (req, res) => {
  res.send(JSON.stringify(req.oidc.user));
});

// Access token for API calls
app.get('/api/data', requiresAuth(), async (req, res) => {
  const { access_token } = await req.oidc.accessToken.get();
  // Use access_token to call API
});
```

### SDK Configuration Options

#### Common Options

| Option | Description | Example |
|--------|-------------|---------|
| `domain` | Auth0 tenant domain | `your-tenant.auth0.com` |
| `clientId` | Application client ID | `abc123...` |
| `redirect_uri` | Callback URL after login | `https://app.com/callback` |
| `audience` | API identifier | `https://api.myapp.com` |
| `scope` | OAuth scopes | `openid profile email` |

#### SPA-Specific Options

| Option | Description | Default |
|--------|-------------|---------|
| `useRefreshTokens` | Use refresh tokens | `false` |
| `cacheLocation` | Token storage | `memory` |
| `useRefreshTokensFallback` | Fallback to iframe | `true` |

```javascript
const auth0 = await createAuth0Client({
  domain: 'your-tenant.auth0.com',
  clientId: 'your-client-id',
  useRefreshTokens: true,
  cacheLocation: 'localstorage', // or 'memory'
  authorizationParams: {
    redirect_uri: window.location.origin,
    audience: 'https://api.myapp.com',
    scope: 'openid profile email offline_access'
  }
});
```

## Application Configuration Best Practices

### Callback URLs
✅ Use HTTPS in production  
✅ Whitelist only necessary URLs  
✅ Be specific (no wildcards if possible)  
✅ Include all environments (dev, staging, prod)  

### Token Settings
✅ Shortest lifetime practical  
✅ Enable refresh token rotation  
✅ Configure appropriate scopes  
✅ Use audience for API authorization  

### Security
✅ Enable PKCE for public clients  
✅ Rotate client secrets regularly (M2M)  
✅ Use state parameter (CSRF protection)  
✅ Implement proper CORS  
✅ Monitor authentication logs  

### UX
✅ Custom domain for branding  
✅ Customize Universal Login  
✅ Clear error messages  
✅ Social login for B2C  
✅ SSO for B2B  

## Key Exam Takeaways

✅ **Application types**: Regular Web, SPA, Native, M2M  
✅ **Regular Web**: Server-side, confidential client, uses client secret, Authorization Code Flow  
✅ **SPA**: Public client, PKCE required, no client secret, JavaScript frameworks  
✅ **Native**: Mobile/desktop, public client, PKCE required, secure storage  
✅ **M2M**: Server-to-server, confidential client, Client Credentials Flow, no user  
✅ **B2C**: Consumers, social login, self-service, large scale  
✅ **B2B**: Business customers, SSO/SAML, multi-tenant, organizations  
✅ **B2E**: Employees, corporate IdP, AD/LDAP, strong security  
✅ **Public clients**: SPA, Native - cannot store secrets, use PKCE  
✅ **Confidential clients**: Regular Web, M2M - can store secrets  
✅ **PKCE**: Required for SPAs and Native apps, prevents code interception  
✅ **Client Credentials Flow**: M2M only, no user context  
✅ **Auth0 Organizations**: Multi-tenancy feature for B2B SaaS  
✅ **Home Realm Discovery**: Route users to correct IdP by email domain  
✅ **Organization Roles**: Scoped to specific organization, different from global roles  
✅ **Create Organizations**: Via Dashboard or Management API with name, branding, metadata  
✅ **Assign Roles to Members**: Add members first, then assign organization-specific roles  
✅ **Organization Invitations**: Invite users by email, can assign roles during invitation  
✅ **org_id in tokens**: Always present when user logs in with organization context  
✅ **Organization vs Regular Roles**: Organization roles are per-org, regular roles are global  
✅ **Member management**: Add via Dashboard or POST /api/v2/organizations/{org_id}/members  
✅ **Role assignment**: POST /api/v2/organizations/{org_id}/members/{user_id}/roles  
✅ **User in multiple orgs**: Can have different roles in different organizations  
✅ **Organization-specific connections**: Each org can have its own SSO provider  
✅ **Validate org membership**: Always check org_id in token matches expected organization  
