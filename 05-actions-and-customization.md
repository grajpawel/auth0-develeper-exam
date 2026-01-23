# Auth0 Actions and Customization

## Auth0 Actions

### What are Actions?

- **Custom Node.js functions** that extend Auth0
- Execute during specific **triggers** in authentication flows
- Replace legacy Rules and Hooks
- Written in JavaScript (Node.js 18)
- Can call external APIs, modify tokens, block logins, etc.

### Key Characteristics

#### Runtime Specifications
- **Node.js version**: 18.x
- **Maximum execution time**: 20 seconds (can timeout)
- **Maximum size**: 
  - Code: 500 KB per action
  - Total size including dependencies: 5 MB
  - Secrets: 100 per action
  - Dependencies: npm modules allowed

#### Performance Limits
- **Timeout**: 20 seconds
- After timeout → Error, flow may fail
- Keep Actions lightweight and fast
- Use async/await for external calls

### Action Triggers

Actions execute at specific points in authentication flows:

#### 1. Login Flow Triggers

##### `post-login`
- **When**: After user authenticates, before token issuance
- **Use cases**:
  - Add custom claims to tokens
  - Enforce MFA based on conditions
  - Enrich user profile
  - Block login based on business logic
  - Account linking
  - Call external APIs for validation

##### `credentials-exchange`
- **When**: During Machine-to-Machine (M2M) Client Credentials flow
- **Use cases**:
  - Add custom claims to M2M access tokens
  - Validate client
  - Modify scopes

##### `pre-user-registration`
- **When**: Before user is created in database
- **Use cases**:
  - Validate user data
  - Block registration based on email domain
  - Add custom metadata
  - Call external validation APIs

##### `post-user-registration`
- **When**: After user is created
- **Use cases**:
  - Send welcome email
  - Create user in external system
  - Add user to groups
  - Log registration event

##### `post-change-password`
- **When**: After password change
- **Use cases**:
  - Send notification email
  - Log event
  - Revoke sessions
  - Update external systems

##### `send-phone-message`
- **When**: Before sending MFA or passwordless SMS
- **Use cases**:
  - Use custom SMS provider
  - Customize message content
  - Add rate limiting
  - Block based on phone number

#### 2. User Profile Triggers

##### `pre-user-registration` (already listed above)

##### `post-user-registration` (already listed above)

#### 3. Multi-factor Authentication Triggers

##### `send-phone-message` (already listed above)

### Action API

#### `event` Object
Contains information about the authentication:
- `event.user` - User profile
- `event.client` - Application details
- `event.connection` - Connection used
- `event.request` - Request metadata (IP, user agent, etc.)
- `event.transaction` - Transaction context

#### `api` Object
Methods to modify the flow:
- `api.access.deny(reason)` - Block authentication
- `api.idToken.setCustomClaim(name, value)` - Add claim to ID token
- `api.accessToken.setCustomClaim(name, value)` - Add claim to access token
- `api.user.setUserMetadata(name, value)` - Update user metadata
- `api.user.setAppMetadata(name, value)` - Update app metadata
- `api.multifactor.enable(provider)` - Require MFA

#### Example: Add Custom Claims
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Add custom claim to access token
  const namespace = 'https://my-app.com';
  api.accessToken.setCustomClaim(`${namespace}/roles`, event.user.app_metadata.roles);
  
  // Add to ID token
  api.idToken.setCustomClaim(`${namespace}/roles`, event.user.app_metadata.roles);
};
```

#### Example: Conditional MFA
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Require MFA for admin users
  if (event.user.app_metadata && event.user.app_metadata.roles.includes('admin')) {
    api.multifactor.enable('any', { allowRememberBrowser: false });
  }
  
  // Require MFA for high-risk IPs
  const highRiskCountries = ['XX', 'YY'];
  if (highRiskCountries.includes(event.request.geoip.countryCode)) {
    api.multifactor.enable('any');
  }
};
```

#### Example: Block Login
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Block unverified emails
  if (!event.user.email_verified) {
    api.access.deny('Please verify your email before logging in.');
  }
  
  // Block based on business logic
  const response = await axios.get(`https://api.example.com/users/${event.user.user_id}/status`);
  if (response.data.blocked) {
    api.access.deny('Your account has been suspended.');
  }
};
```

### Secrets and Dependencies

#### Secrets
- Store sensitive data (API keys, tokens)
- Encrypted at rest
- Accessed via `event.secrets.SECRET_NAME`
- Maximum 100 secrets per action
- Configured in Action editor

#### Dependencies
- Use npm packages
- Specify in Action editor
- Loaded at runtime
- Count toward 5 MB total size limit
- Common packages: `axios`, `lodash`, `jsonwebtoken`

### Action Flows

Actions are organized into **flows** (ordered sequences):

#### Creating a Flow
1. Actions → Library → Create custom action
2. Select trigger type
3. Write code
4. Add to flow (drag and drop)
5. Set execution order
6. Deploy

#### Flow Execution
- Actions execute in order (top to bottom)
- If one action denies access, flow stops
- All actions must complete within timeout

### Testing Actions

#### Built-in Testing
- Test runner in Action editor
- Simulate events
- View console logs
- Check execution time

#### Real-time Webtask Logs
- View logs in Auth0 Dashboard
- Monitor → Logs
- Filter by action name
- See console.log output

### Migration from Rules

#### Rules (Legacy) vs Actions

| Rules | Actions |
|-------|---------|
| Single trigger (login) | Multiple triggers |
| Global scope | Flow-specific |
| Deprecated | Current standard |
| Execute in order | Flow-based execution |

#### Key Differences
- Actions have explicit triggers
- Better error handling in Actions
- Actions support secrets natively
- Scoped to specific flows

## Custom Domains

### Configuring a Custom Domain for Universal Login (CCDUL)

#### Why Custom Domains?

**Benefits**:
- ✅ Branded experience (`login.yourcompany.com` vs `yourcompany.auth0.com`)
- ✅ Better user trust
- ✅ Consistent domain across experience
- ✅ Required for some enterprise customers
- ✅ Avoid third-party cookie issues

#### Setup Process

1. **Choose Domain**
   - Subdomain recommended: `login.example.com`, `auth.example.com`
   - Must own the domain

2. **Configure in Auth0**
   - Dashboard → Branding → Custom Domains
   - Enter your domain
   - Choose verification method

3. **Verify Domain Ownership**
   - **Option A**: DNS TXT record
   - **Option B**: DNS CNAME record
   - Add record to DNS provider
   - Wait for verification (can take minutes to hours)

4. **Configure SSL Certificate**
   - **Auth0 Managed** (recommended):
     - Auth0 provisions certificate automatically
     - Auto-renewal
     - Free
   - **Self-Managed**:
     - Upload your own certificate
     - Handle renewal yourself

5. **Update DNS**
   - Add CNAME record pointing to Auth0
   - `login.example.com` → `yourcompany.edge.tenants.auth0.com`

6. **Update Application Settings**
   - Update callback URLs to use custom domain
   - Update logout URLs
   - Update allowed origins

#### Verification Methods

##### TXT Record
- Add TXT record with verification code
- Quick verification
- Can be removed after verification

##### CNAME Record
- Add CNAME pointing to Auth0
- Slower verification
- Required for final setup anyway

#### Certificate Options

##### Auth0 Managed (Recommended)
- ✅ Automatic provisioning
- ✅ Auto-renewal
- ✅ No cost
- ✅ No maintenance

##### Self-Managed
- Upload your own certificate
- Must be PEM format
- Include full chain
- Renew before expiration

## Error Pages

### Auth0 Error Pages

#### When Are They Displayed?

Error pages appear when:
- Login fails (invalid credentials)
- Authorization denied
- MFA enrollment fails
- Token exchange fails
- Action denies access
- Session expired
- Invalid callback URL
- Consent denied
- Rate limiting triggered

#### Default Error Page

Auth0 provides default generic error page:
- Basic styling
- Error code and description
- "Go back" link
- Not branded

### Customizing Error Pages (CEP)

#### How to Customize

1. **Dashboard → Branding → Universal Login → Advanced Options**
2. **Click "Error Page" tab**
3. **Enable custom error page**
4. **Edit HTML template**
5. **Save changes**

#### Template Variables

Available in error page template:
- `@@CONFIG@@` - Configuration object (JSON)
- Error details in config:
  - `error` - Error code
  - `error_description` - Human-readable description
  - `name` - User-friendly error name

#### Example Custom Error Page
```html
<!DOCTYPE html>
<html>
<head>
  <title>Authentication Error</title>
  <style>
    body { font-family: Arial; text-align: center; padding: 50px; }
    .error-code { color: #d32f2f; font-size: 24px; }
  </style>
</head>
<body>
  <h1>Oops! Something went wrong</h1>
  <div class="error-code">Error: ${error}</div>
  <p>${errorDescription}</p>
  <script>
    var config = JSON.parse(decodeURIComponent(escape(window.atob('@@CONFIG@@'))));
    document.querySelector('.error-code').textContent = 'Error: ' + config.error;
    document.querySelector('p').textContent = config.error_description;
  </script>
</body>
</html>
```

#### Common Customizations
- Match brand colors and fonts
- Add company logo
- Provide helpful error messages
- Add contact information
- Include navigation back to main site
- Localization for multiple languages

### Error Handling Best Practices

✅ User-friendly error messages (avoid technical jargon)  
✅ Provide actionable next steps  
✅ Include support contact information  
✅ Log errors for debugging  
✅ Match branding consistently  
✅ Handle all error scenarios  
✅ Test error pages thoroughly  

## Universal Login Customization

### What is Universal Login?

- Auth0-hosted login page
- Centralized authentication experience
- Recommended approach (vs embedded login)
- Consistent security updates
- Customizable branding

### Customization Options

#### 1. Simple Customization
- Logo
- Primary color
- Background color
- Configure in Dashboard → Branding → Universal Login

#### 2. Advanced Customization
- Edit HTML/CSS/JavaScript
- Full control over appearance
- Use custom templates
- Access via Dashboard → Branding → Universal Login → Advanced

#### 3. Custom Domain
- Use your own domain (covered above)
- Fully branded URL

## Key Exam Takeaways

✅ **Actions**: Node.js 18, max 20 sec timeout, max 5 MB total size  
✅ **Action triggers**: post-login, pre-user-registration, post-user-registration, credentials-exchange, etc.  
✅ **post-login**: Most common, add claims, enforce MFA, block login  
✅ **credentials-exchange**: For M2M flows only  
✅ **Actions API**: api.access.deny(), api.idToken.setCustomClaim(), api.multifactor.enable()  
✅ **Secrets**: Max 100 per action, stored encrypted  
✅ **Custom domains**: Improves branding, requires DNS CNAME, Auth0 can manage SSL  
✅ **Error pages**: Shown on auth failures, customizable via Dashboard  
✅ **Custom error pages**: Edit HTML template, use @@CONFIG@@ for error details  
✅ **Action size limits**: 500 KB code, 5 MB total with dependencies  
✅ **Action timeout**: 20 seconds, then fails  
✅ **Custom domain benefits**: Branding, trust, avoid cookie issues  
✅ **Custom domain setup**: Domain verification → SSL cert → DNS CNAME → Update callbacks  
