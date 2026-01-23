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
