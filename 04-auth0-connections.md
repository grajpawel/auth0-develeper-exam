# Auth0 Connections

## What are Connections?

Connections are **identity sources** that Auth0 can authenticate users against. They define how and where users authenticate.

### Connection Types Overview

1. **Database Connections** - Auth0-hosted user storage
2. **Social Connections** - OAuth providers (Google, Facebook, etc.)
3. **Enterprise Connections** - Corporate identity providers (SAML, OIDC, AD)
4. **Passwordless Connections** - Email or SMS-based authentication

## Database Connections

### What are Database Connections?

- Auth0 stores and manages user credentials
- Username/email + password authentication
- Full control over user data
- Most common for B2C applications

### Features

#### User Storage
- Stored securely in Auth0 database
- Passwords hashed with bcrypt
- PII encrypted at rest
- Compliance-ready (GDPR, SOC2, etc.)

#### Password Policies
Configurable requirements:
- Minimum length
- Character requirements (uppercase, numbers, symbols)
- Password history (prevent reuse)
- Dictionary/breach detection
- Custom password validation

#### Sign-up Configuration
- Disable sign-ups (invitation-only)
- Require email/username
- Custom username validation
- Auto-login after signup

#### Import Users
- Bulk import from other systems
- Custom database scripts
- Lazy migration (import on login)
- Automatic password migration

### Custom Database Scripts

Connect to your existing user database instead of migrating:

#### Available Scripts
1. **Login** - Authenticate user against your DB
2. **Create** - Create user in your DB
3. **Verify** - Verify user email
4. **Change Password** - Update password
5. **Get User** - Retrieve user profile
6. **Delete** - Remove user

#### Progressive Migration
- Keep users in your DB initially
- Users authenticate against your DB
- Auth0 stores password hash on first login
- Eventually migrate all users to Auth0

#### Use Cases
- Large existing user base
- Can't migrate all users at once
- Need to keep legacy system running
- Gradual migration approach

## Social Connections

### What are Social Connections?

- Authentication via social identity providers
- OAuth 2.0 based
- No password management needed
- Popular for B2C consumer apps

### Supported Providers

#### Major Providers
- Google
- Facebook
- Apple
- Microsoft
- LinkedIn
- GitHub
- Twitter/X

#### Features
- **Default Apps**: Auth0 provides developer keys (for testing)
- **Custom Apps**: Use your own OAuth app credentials (production)
- **Permissions**: Request specific user data (email, profile, etc.)

### Configuration

#### Using Auth0 Dev Keys
- Quick setup for development
- Shared among all Auth0 customers
- **Not for production** (rate limits, branding)

#### Using Your Own Keys
1. Create OAuth app in provider (Google Cloud, Facebook Developers, etc.)
2. Configure redirect URLs
3. Add client ID and secret to Auth0
4. Request appropriate scopes

#### Requested Scopes
- `email` - User's email address
- `profile` - Basic profile info
- Provider-specific scopes (e.g., Google Calendar access)

### User Profile

- Auth0 normalizes user data across providers
- Standard fields: `email`, `name`, `picture`
- Provider-specific data in `identities` array
- Can merge accounts from multiple social providers

## Auth0 Normalized User Profile

### What is the Normalized User Profile?

Auth0 normalizes user data from different identity providers into a **consistent, standardized format**. This means regardless of whether a user logs in with Google, Facebook, SAML, or a database connection, your application receives user data in the same structure.

### Why Normalization Matters

**Without Normalization**:
- Google returns `given_name` and `family_name`
- Facebook returns `first_name` and `last_name`
- SAML might return `FirstName` and `LastName`
- Your app needs to handle every variation

**With Auth0 Normalization**:
- All providers map to `name`, `given_name`, `family_name`
- Consistent structure regardless of source
- Simplified application code
- Single schema to work with

### Normalized Profile Structure

#### Core Attributes (Always Present)

| Attribute | Type | Description |
|-----------|------|-------------|
| `user_id` | String | Unique identifier (e.g., `auth0|507f1f77bcf86cd799439020`) |
| `email` | String | User's email address |
| `email_verified` | Boolean | Whether email has been verified |
| `name` | String | Full name |
| `picture` | String | URL to profile picture |
| `nickname` | String | User's nickname or username |
| `created_at` | String | Account creation timestamp |
| `updated_at` | String | Last profile update timestamp |

#### Optional Attributes (Provider-Dependent)

| Attribute | Type | Description |
|-----------|------|-------------|
| `given_name` | String | First name |
| `family_name` | String | Last name |
| `phone_number` | String | Phone number |
| `phone_verified` | Boolean | Phone verification status |
| `locale` | String | User's locale (e.g., `en-US`) |
| `username` | String | Username for database connections |

### Identities Array

The `identities` array contains information about all linked accounts:

```json
{
  "user_id": "auth0|507f1f77bcf86cd799439020",
  "email": "user@example.com",
  "name": "John Doe",
  "picture": "https://example.com/photo.jpg",
  "identities": [
    {
      "provider": "auth0",
      "user_id": "507f1f77bcf86cd799439020",
      "connection": "Username-Password-Authentication",
      "isSocial": false
    },
    {
      "provider": "google-oauth2",
      "user_id": "115015401343387192503",
      "connection": "google-oauth2",
      "isSocial": true
    }
  ]
}
```

#### Identity Object Properties

| Property | Description |
|----------|-------------|
| `provider` | Identity provider name |
| `user_id` | User ID from that provider |
| `connection` | Auth0 connection name |
| `isSocial` | Whether it's a social provider |
| `access_token` | Provider's access token (if stored) |
| `profileData` | Raw profile data from provider |

### Provider-Specific Mapping

#### Google
```json
// Google returns:
{
  "sub": "115015401343387192503",
  "email": "user@gmail.com",
  "given_name": "John",
  "family_name": "Doe",
  "picture": "https://lh3.googleusercontent.com/..."
}

// Auth0 normalizes to:
{
  "user_id": "google-oauth2|115015401343387192503",
  "email": "user@gmail.com",
  "name": "John Doe",
  "given_name": "John",
  "family_name": "Doe",
  "picture": "https://lh3.googleusercontent.com/...",
  "nickname": "user"
}
```

#### Facebook
```json
// Facebook returns:
{
  "id": "10157820145678901",
  "email": "user@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "picture": { "data": { "url": "https://..." } }
}

// Auth0 normalizes to:
{
  "user_id": "facebook|10157820145678901",
  "email": "user@example.com",
  "name": "John Doe",
  "given_name": "John",
  "family_name": "Doe",
  "picture": "https://graph.facebook.com/...",
  "nickname": "john.doe"
}
```

#### SAML/Enterprise
```xml
<!-- SAML Assertion -->
<Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress">
  <AttributeValue>user@company.com</AttributeValue>
</Attribute>
<Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname">
  <AttributeValue>John</AttributeValue>
</Attribute>
```

```json
// Auth0 normalizes to:
{
  "user_id": "samlp|company-idp|john@company.com",
  "email": "user@company.com",
  "name": "John Doe",
  "given_name": "John",
  "family_name": "Doe"
}
```

### User ID Format

The `user_id` follows a specific format:
```
{identity-provider}|{unique-id}
```

**Examples**:
- `auth0|507f1f77bcf86cd799439020` - Database connection
- `google-oauth2|115015401343387192503` - Google
- `facebook|10157820145678901` - Facebook
- `samlp|company-idp|user@company.com` - SAML
- `windowslive|user@outlook.com` - Microsoft

### Accessing Normalized Profile

#### In ID Token
```json
{
  "sub": "auth0|507f1f77bcf86cd799439020",
  "name": "John Doe",
  "email": "user@example.com",
  "email_verified": true,
  "picture": "https://example.com/photo.jpg",
  "nickname": "johnd",
  "updated_at": "2024-01-15T10:30:00.000Z"
}
```

#### Via Management API
```javascript
GET /api/v2/users/{user_id}

// Response
{
  "user_id": "auth0|507f1f77bcf86cd799439020",
  "email": "user@example.com",
  "email_verified": true,
  "name": "John Doe",
  "picture": "https://example.com/photo.jpg",
  "nickname": "johnd",
  "identities": [...],
  "user_metadata": {...},
  "app_metadata": {...}
}
```

#### Via /userinfo Endpoint
```javascript
GET /userinfo
Authorization: Bearer {access_token}

// Response (OIDC standard claims)
{
  "sub": "auth0|507f1f77bcf86cd799439020",
  "name": "John Doe",
  "email": "user@example.com",
  "email_verified": true,
  "picture": "https://example.com/photo.jpg"
}
```

### User Metadata vs App Metadata

#### User Metadata (`user_metadata`)
- **Editable by user** (via profile forms)
- Accessible to the user
- Preferences, settings, personal info
- Examples: timezone, language preference, theme

```json
"user_metadata": {
  "preferences": {
    "theme": "dark",
    "language": "en",
    "timezone": "America/New_York"
  },
  "profile": {
    "company": "Acme Corp",
    "job_title": "Developer"
  }
}
```

**Updating via SDK**:
```javascript
// User can update their own metadata
await auth0.patchUserAttributes(userId, {
  user_metadata: {
    preferences: { theme: 'light' }
  }
});
```

#### App Metadata (`app_metadata`)
- **Only editable by application** (via Management API)
- **Not accessible to user** (hidden from profile)
- Authorization data, internal flags
- Examples: plan type, roles, internal IDs

```json
"app_metadata": {
  "plan": "enterprise",
  "roles": ["admin", "billing"],
  "signup_source": "marketing_campaign",
  "stripe_customer_id": "cus_abc123",
  "organization_id": "org_xyz789",
  "permissions": ["read:reports", "write:users"],
  "blocked": false,
  "mfa_required": true
}
```

**Updating via Management API**:
```javascript
// Only application can update app_metadata
await management.users.update(
  { id: userId },
  { app_metadata: { plan: 'premium' } }
);
```

#### Metadata Comparison

| Feature | user_metadata | app_metadata |
|---------|--------------|--------------|
| **Who can edit** | User or application | Application only |
| **Visible to user** | Yes | No |
| **Use case** | Preferences, settings | Authorization, internal data |
| **Size limit** | 16 KB | 16 KB |
| **In ID token by default** | No | No |
| **Access in Actions** | `event.user.user_metadata` | `event.user.app_metadata` |

#### Metadata Limits

| Limit | Value |
|-------|-------|
| `user_metadata` size | 16 KB max |
| `app_metadata` size | 16 KB max |
| Total user profile | 64 KB max |
| Nested depth | Recommended: 3 levels |
| Field name length | 255 characters |

**Important**: Metadata is **not searchable** by default. For searchable data, consider using a separate database.

### Complete User Profile Structure

```json
{
  // Core attributes (always present)
  "user_id": "auth0|507f1f77bcf86cd799439020",
  "email": "user@example.com",
  "email_verified": true,
  "name": "John Doe",
  "nickname": "johnd",
  "picture": "https://example.com/photo.jpg",
  "created_at": "2024-01-01T00:00:00.000Z",
  "updated_at": "2024-01-15T10:30:00.000Z",
  
  // Optional attributes (provider-dependent)
  "given_name": "John",
  "family_name": "Doe",
  "phone_number": "+1234567890",
  "phone_verified": true,
  "username": "johndoe",
  "locale": "en-US",
  
  // Login tracking
  "last_login": "2024-01-15T10:30:00.000Z",
  "last_ip": "192.168.1.1",
  "logins_count": 42,
  
  // Identities (linked accounts)
  "identities": [
    {
      "provider": "auth0",
      "user_id": "507f1f77bcf86cd799439020",
      "connection": "Username-Password-Authentication",
      "isSocial": false
    },
    {
      "provider": "google-oauth2",
      "user_id": "115015401343387192503",
      "connection": "google-oauth2",
      "isSocial": true,
      "profileData": {
        "email": "user@gmail.com",
        "email_verified": true,
        "name": "John Doe",
        "given_name": "John",
        "family_name": "Doe",
        "picture": "https://lh3.googleusercontent.com/..."
      }
    }
  ],
  
  // Custom metadata
  "user_metadata": {
    "preferences": { "theme": "dark" }
  },
  "app_metadata": {
    "roles": ["admin"],
    "plan": "enterprise"
  },
  
  // MFA (if enrolled)
  "multifactor": ["guardian"],
  
  // Blocked status
  "blocked": false
}
```

### Fields Available in Different Contexts

| Field | ID Token | /userinfo | Management API | Actions |
|-------|----------|-----------|----------------|---------|
| `sub` / `user_id` | ✅ | ✅ | ✅ | ✅ |
| `email` | ✅ | ✅ | ✅ | ✅ |
| `email_verified` | ✅ | ✅ | ✅ | ✅ |
| `name` | ✅ | ✅ | ✅ | ✅ |
| `picture` | ✅ | ✅ | ✅ | ✅ |
| `nickname` | ✅ | ✅ | ✅ | ✅ |
| `given_name` | ✅* | ✅* | ✅ | ✅ |
| `family_name` | ✅* | ✅* | ✅ | ✅ |
| `identities` | ❌ | ❌ | ✅ | ✅ |
| `user_metadata` | ❌** | ❌ | ✅ | ✅ |
| `app_metadata` | ❌** | ❌ | ✅ | ✅ |
| `logins_count` | ❌ | ❌ | ✅ | ✅ |
| `last_login` | ❌ | ❌ | ✅ | ✅ |
| `last_ip` | ❌ | ❌ | ✅ | ✅ |
| `blocked` | ❌ | ❌ | ✅ | ✅ |

*Requires `profile` scope  
**Can be added via Actions using `setCustomClaim()`
```

### Customizing Profile with Actions

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Add custom claims from normalized profile
  api.idToken.setCustomClaim('https://myapp.com/email', event.user.email);
  
  // Access identities
  const identities = event.user.identities || [];
  const providers = identities.map(i => i.provider);
  api.idToken.setCustomClaim('https://myapp.com/providers', providers);
  
  // Access metadata
  const plan = event.user.app_metadata?.plan || 'free';
  api.accessToken.setCustomClaim('https://myapp.com/plan', plan);
};
```

### Attribute Mapping for Enterprise

For SAML/OIDC enterprise connections, configure attribute mapping:

**Dashboard Configuration**:
1. Go to **Authentication** → **Enterprise**
2. Select connection
3. Go to **Mappings** tab
4. Map IdP attributes to Auth0 profile

**Common Mappings**:
```json
{
  "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
  "name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
  "given_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
  "family_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname",
  "groups": "http://schemas.microsoft.com/ws/2008/06/identity/claims/groups"
}
```

### Best Practices

✅ **Use normalized attributes** in your application
- Don't rely on provider-specific fields
- Access `name`, `email`, `picture` consistently

✅ **Store custom data in metadata**
- User preferences → `user_metadata`
- Application data → `app_metadata`
- Don't modify core profile attributes

✅ **Handle missing attributes gracefully**
- Not all providers return all fields
- Check for null/undefined values
- Provide default values

✅ **Use Actions for custom claims**
- Enrich tokens with additional data
- Map metadata to token claims
- Keep tokens minimal

✅ **Configure enterprise attribute mapping**
- Ensure SAML/OIDC attributes map correctly
- Test with sample assertions
- Document mapping decisions

## Enterprise Connections

### What are Enterprise Connections?

- Integration with corporate identity providers
- Used for B2B and employee applications
- Supports SSO (Single Sign-On)
- Common protocols: SAML, OIDC, LDAP

### Connection Types

#### SAML
- **Security Assertion Markup Language**
- XML-based protocol
- Enterprise standard
- Providers: Okta, OneLogin, Azure AD, Ping Identity

#### OpenID Connect (OIDC)
- Modern JSON-based protocol
- OAuth 2.0 extension
- Easier to implement than SAML
- Providers: Azure AD, Google Workspace, Okta

#### LDAP / Active Directory
- Direct integration with AD/LDAP servers
- Requires AD/LDAP connector (on-premises agent)
- Kerberos support
- Windows Integrated Authentication

#### Azure Active Directory
- Native integration with Microsoft's IdP
- Supports Office 365 users
- Multiple connection options (OIDC, SAML, WS-Fed)

### Configuration

#### SAML Setup
1. Obtain IdP metadata (XML)
2. Configure in Auth0 (upload metadata or manual)
3. Provide Auth0 metadata to IdP
4. Map attributes (email, name, groups)
5. Test connection

#### OIDC Setup
1. Register application with IdP
2. Get client ID, secret, and endpoints
3. Configure in Auth0
4. Map claims
5. Test connection

#### Domain-based Routing
- Home Realm Discovery (HRD)
- Route users based on email domain
- E.g., `@company.com` → Company SAML connection
- Configurable in Auth0

## Passwordless Connections

### What is Passwordless?

- Authentication without passwords
- Uses email or SMS
- One-time codes or magic links
- Reduces password fatigue

### Types

#### Email Passwordless
1. **Magic Link**
   - User enters email
   - Receives link via email
   - Clicks link to authenticate
   - No code to type

2. **Email Code**
   - User enters email
   - Receives 6-digit code
   - Enters code to authenticate
   - Code expires quickly (typically 5 min)

#### SMS Passwordless
- User enters phone number
- Receives SMS with code
- Enters code to authenticate
- Requires SMS provider (Twilio, etc.)

### Configuration

#### Email Setup
- Use Auth0's email provider (limited) or custom (SendGrid, Mailgun, etc.)
- Customize email templates
- Choose link vs code
- Set code expiration

#### SMS Setup
- Requires Twilio, Vonage, or custom provider
- Configure provider credentials
- Customize message template
- Set code expiration
- Rate limiting to prevent abuse

### Use Cases

✅ **Good for**:
- Consumer apps (B2C)
- Reduced friction
- No password reset flow needed
- Mobile-first apps

⚠️ **Consider**:
- Email/SMS delivery reliability
- Cost (SMS can be expensive)
- Less secure than MFA (single factor)
- Potential for SIM swapping (SMS)

## Connection Selection

### When to Use Each Type

#### Database Connections
- ✅ B2C applications
- ✅ Full control over user data
- ✅ Custom password policies
- ✅ When no existing identity provider

#### Social Connections
- ✅ Consumer apps
- ✅ Quick sign-up
- ✅ Reduce friction
- ✅ Target audience uses social media

#### Enterprise Connections
- ✅ B2B applications
- ✅ Corporate customers
- ✅ SSO requirement
- ✅ Integration with existing IdP

#### Passwordless
- ✅ Mobile-first apps
- ✅ Maximum convenience
- ✅ No password management
- ⚠️ Consider adding MFA for security

### Multiple Connections

Applications can enable multiple connections:
- Database + Social (common for B2C)
- Multiple enterprise connections (multi-tenant B2B)
- Passwordless + Database (fallback)

Users see connection selector at login (or HRD based on email domain).

## Account Linking

### What is Account Linking?

Merging multiple user identities into single account:
- User has both Google and Facebook login → Link them
- Same email address across connections
- Single user profile with multiple authentication methods

### Linking Methods

#### 1. Automatic Account Linking
- Not available by default (security risk)
- Can implement via Auth0 Actions
- Link accounts with matching email addresses
- **Risk**: Email hijacking can lead to account takeover

#### 2. Prompt-based Account Linking
- User logs in with new provider
- Auth0 detects existing account with same email
- Prompts user to link accounts
- User authenticates with original provider to confirm
- **Most secure** option

#### 3. Manual Account Linking
- User initiates from profile/settings page
- Application calls Auth0 Management API
- Links secondary identity to primary account
- User remains in control

### Implementation

#### Using Auth0 Management API
```javascript
// Link accounts
PATCH /api/v2/users/{primary_user_id}
{
  "link_with": "secondary_user_id"
}
```

#### Using Actions
- Trigger: `post-login`
- Check for existing user with same email
- Prompt or automatically link
- Update user metadata

### Best Practices

✅ Require confirmation before linking  
✅ Verify email addresses  
✅ Notify users when accounts are linked  
✅ Allow users to unlink accounts  
✅ Audit account linking events  
⚠️ Never auto-link without verification (security risk)  

## User Provisioning

### Just-In-Time (JIT) Provisioning

#### What is JIT Provisioning?

**Just-In-Time provisioning** automatically creates user accounts in Auth0 when users first authenticate through an enterprise IdP (SAML, OIDC):

- User doesn't need to exist in Auth0 beforehand
- Account created on first SSO login
- Profile data populated from IdP assertions

#### How JIT Works

```
User first login attempt via SAML/OIDC
    ↓
IdP authenticates user
    ↓
IdP sends assertion to Auth0
    ↓
Auth0 checks: Does user exist?
    ↓
No → Create user with IdP attributes
    ↓
Auth0 completes authentication
```

#### JIT Configuration

**Enabled by default** for enterprise connections. Configure attribute mapping to populate profile:

**Dashboard → Authentication → Enterprise → [Connection] → Mappings**

```json
{
  "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
  "name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
  "given_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
  "family_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname",
  "groups": "http://schemas.microsoft.com/ws/2008/06/identity/claims/groups"
}
```

#### JIT with Actions

Enhance JIT with post-login Actions:

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Check if this is first login (JIT created user)
  if (event.stats.logins_count === 1) {
    // First login - send welcome email
    await sendWelcomeEmail(event.user.email);
    
    // Set initial metadata
    api.user.setAppMetadata('onboarded', false);
    api.user.setAppMetadata('signup_source', 'enterprise_sso');
    
    // Assign default role (would use Management API)
    api.accessToken.setCustomClaim('https://myapp.com/new_user', true);
  }
};
```

#### JIT Benefits

✅ **No pre-provisioning** required  
✅ **Reduced admin work** (no manual user creation)  
✅ **Profile sync** on each login  
✅ **Self-service** for enterprise users  

#### JIT Limitations

❌ No user exists until first login  
❌ Can't assign roles/permissions before first login  
❌ Limited to IdP-provided attributes  
❌ No deprovisioning (user removal)  

### SCIM (System for Cross-domain Identity Management)

#### What is SCIM?

**SCIM** is a standardized protocol for **automated user provisioning and deprovisioning**:

- Create users before they log in
- Update user attributes automatically
- Deactivate/delete users when they leave
- Sync groups/roles from IdP

#### SCIM vs JIT Provisioning

| Feature | JIT Provisioning | SCIM |
|---------|-----------------|------|
| **User Creation** | On first login | Before first login |
| **Pre-provisioning** | ❌ No | ✅ Yes |
| **Deprovisioning** | ❌ No | ✅ Yes |
| **Attribute Updates** | On login only | Real-time sync |
| **Group Sync** | Limited | ✅ Full support |
| **Complexity** | Simple | More complex |
| **Use Case** | Basic SSO | Full lifecycle |

#### SCIM in Auth0

Auth0 supports SCIM for **inbound provisioning** (IdP → Auth0):

**Supported IdPs**:
- Okta
- Azure AD
- OneLogin
- Ping Identity
- Other SCIM 2.0 compliant providers

#### SCIM Setup

**Dashboard → Authentication → Enterprise → [Connection] → Provisioning**

1. Enable SCIM provisioning
2. Generate SCIM endpoint URL
3. Generate bearer token
4. Configure IdP with Auth0 SCIM endpoint

**Auth0 SCIM Endpoint**:
```
https://{tenant}.auth0.com/scim/v2/{connection_id}
```

#### SCIM Operations

##### Create User
```http
POST /scim/v2/{connection_id}/Users
Authorization: Bearer {token}
Content-Type: application/scim+json

{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "userName": "john.doe@example.com",
  "name": {
    "givenName": "John",
    "familyName": "Doe"
  },
  "emails": [{
    "value": "john.doe@example.com",
    "primary": true
  }],
  "active": true
}
```

##### Update User
```http
PATCH /scim/v2/{connection_id}/Users/{user_id}
Authorization: Bearer {token}
Content-Type: application/scim+json

{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [{
    "op": "replace",
    "path": "name.givenName",
    "value": "Jonathan"
  }]
}
```

##### Deactivate User
```http
PATCH /scim/v2/{connection_id}/Users/{user_id}
Authorization: Bearer {token}
Content-Type: application/scim+json

{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [{
    "op": "replace",
    "path": "active",
    "value": false
  }]
}
```

##### Delete User
```http
DELETE /scim/v2/{connection_id}/Users/{user_id}
Authorization: Bearer {token}
```

#### SCIM Attribute Mapping

Map SCIM attributes to Auth0 profile:

```json
{
  "user_id": "externalId",
  "email": "emails[type eq \"work\"].value",
  "name": "displayName",
  "given_name": "name.givenName",
  "family_name": "name.familyName",
  "app_metadata.department": "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department"
}
```

#### Group Provisioning

SCIM can sync groups from IdP:

```http
POST /scim/v2/{connection_id}/Groups
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
  "displayName": "Engineering",
  "members": [
    {"value": "user_id_1"},
    {"value": "user_id_2"}
  ]
}
```

#### SCIM Best Practices

✅ **Use HTTPS** for all SCIM endpoints  
✅ **Secure bearer token** and rotate periodically  
✅ **Map required attributes** properly  
✅ **Test with small group** before full rollout  
✅ **Monitor provisioning logs** for errors  
✅ **Handle deprovisioning** (block vs delete)  

### When to Use Each

#### Use JIT Provisioning When:
- ✅ Simple SSO integration
- ✅ No need for pre-provisioning
- ✅ Deprovisioning not critical
- ✅ Quick implementation needed

#### Use SCIM When:
- ✅ Full user lifecycle management needed
- ✅ Users must exist before first login
- ✅ Automatic deprovisioning required
- ✅ Group/role sync needed
- ✅ Compliance requires account disable on termination

## Key Exam Takeaways

✅ **Four connection types**: Database, Social, Enterprise, Passwordless  
✅ **Database connections**: Auth0 stores credentials, custom password policies  
✅ **Social connections**: OAuth providers, no password management  
✅ **Enterprise connections**: SAML, OIDC, AD/LDAP for B2B/SSO  
✅ **Passwordless**: Email (link/code) or SMS, convenient but single factor  
✅ **Custom database scripts**: Progressive migration, connect to existing DB  
✅ **Account linking methods**: Automatic (risky), Prompt-based (secure), Manual  
✅ **HRD**: Home Realm Discovery, route users by email domain  
✅ **SAML**: XML-based, enterprise standard  
✅ **OIDC**: JSON-based, modern alternative to SAML  
✅ **Social provider keys**: Use your own for production, not Auth0 dev keys  
✅ **AD/LDAP connector**: On-premises agent required for direct AD integration  
✅ **Normalized User Profile**: Auth0 standardizes user data from all providers  
✅ **user_id format**: `{provider}|{unique-id}` (e.g., `google-oauth2|123456`)  
✅ **Identities array**: Contains all linked accounts with provider-specific data  
✅ **Core attributes**: `user_id`, `email`, `name`, `picture`, `nickname` always present  
✅ **user_metadata**: User-editable data (preferences), 16 KB limit  
✅ **app_metadata**: Application-only data (roles, plan), not visible to user, 16 KB limit  
✅ **Metadata in tokens**: Not included by default, add via Actions `setCustomClaim()`  
✅ **Total profile size**: 64 KB max for entire user object  
✅ **Attribute mapping**: Configure for SAML/enterprise connections  
✅ **/userinfo endpoint**: Returns normalized OIDC claims for authenticated user  
✅ **last_login, logins_count**: Tracking fields available in Management API  
✅ **JIT Provisioning**: Auto-create users on first SSO login  
✅ **SCIM**: Standardized protocol for user provisioning/deprovisioning  
✅ **JIT vs SCIM**: JIT = simple/on-login, SCIM = full lifecycle management  
✅ **SCIM operations**: Create, Update, Deactivate, Delete users  
✅ **SCIM deprovisioning**: Can block or delete users when removed from IdP  
✅ **Group sync**: SCIM supports syncing groups from IdP to Auth0  
