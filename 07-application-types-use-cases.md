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
