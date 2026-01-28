# Universal Login

## What is Universal Login?

Universal Login is Auth0's **centralized, hosted login page** that handles authentication for all your applications. Instead of embedding login forms directly in your application, users are redirected to Auth0's hosted page to authenticate.

### Why Universal Login?

**Benefits**:
- ✅ **Single source of truth** for authentication
- ✅ **Security updates** handled by Auth0 automatically
- ✅ **Consistent experience** across all applications
- ✅ **Reduced attack surface** (credentials never enter your app)
- ✅ **Easier compliance** (PCI DSS, HIPAA)
- ✅ **All connection types** work automatically (social, enterprise, database)
- ✅ **MFA built-in** without extra implementation
- ✅ **No cross-origin issues** (first-party cookies)

### Universal Login vs Embedded Login

| Feature | Universal Login | Embedded Login |
|---------|-----------------|----------------|
| **Location** | Auth0-hosted page | Your application |
| **Security** | Higher (credentials on Auth0 domain) | Lower (credentials on your domain) |
| **Maintenance** | Auth0 handles updates | You handle updates |
| **Customization** | Template-based | Full control |
| **Cookie Issues** | None (first-party) | Third-party cookie problems |
| **Social/Enterprise** | All connections work | Limited support |
| **MFA** | Built-in | Manual implementation |
| **Recommendation** | ✅ Recommended | ⚠️ Use only when necessary |

## Classic vs New Universal Login

Auth0 offers two Universal Login experiences:

### Classic Universal Login

**Characteristics**:
- Uses **Lock** widget (pre-built UI component)
- Or custom HTML/CSS/JavaScript
- Legacy approach
- Still supported but not recommended for new projects

**When to Use**:
- Existing applications already using Classic
- Need very specific customizations not available in New
- Legacy compatibility requirements

### New Universal Login (Recommended)

**Characteristics**:
- Modern, optimized experience
- **No-code customization** via Dashboard
- Better performance (faster load times)
- Automatic security updates
- Built on latest best practices

**When to Use**:
- ✅ All new applications
- ✅ Modern branding requirements
- ✅ When migrating from Classic

### Comparison Table

| Feature | Classic Universal Login | New Universal Login |
|---------|------------------------|---------------------|
| **Customization** | Full HTML/CSS/JS control | No-code + limited code |
| **Performance** | Standard | Optimized |
| **Maintenance** | Manual updates | Automatic updates |
| **Lock Widget** | Yes | No (uses native UI) |
| **Branding** | Custom templates | Dashboard settings |
| **Recommendation** | Legacy only | ✅ Use for new projects |

## Universal Login Customization

### Basic Customization (Dashboard)

**Dashboard → Branding → Universal Login**

#### 1. Logo
- Upload company logo
- Displayed on login page
- Recommended size: 150x150px minimum
- Formats: PNG, JPG, SVG

#### 2. Colors
- **Primary Color**: Buttons, links
- **Page Background**: Login page background
- **Widget Background**: Login box background

#### 3. Fonts
- Choose from available fonts
- Or use custom web fonts

### Advanced Customization

#### Page Templates (New Universal Login)

Customize specific elements:
- Login page
- Signup page
- Password reset page
- MFA enrollment
- Error pages

#### Custom HTML (Classic Only)

Full control over login page HTML:
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Login</title>
  <style>
    /* Custom CSS */
    .auth0-lock-header-logo {
      width: 150px;
    }
  </style>
</head>
<body>
  <script src="https://cdn.auth0.com/js/lock/12.0/lock.min.js"></script>
  <script>
    var lock = new Auth0Lock(
      '@@config@@.clientID',
      '@@config@@.auth0Domain',
      {
        theme: {
          logo: 'https://example.com/logo.png',
          primaryColor: '#0066CC'
        },
        languageDictionary: {
          title: 'Welcome'
        }
      }
    );
    lock.show();
  </script>
</body>
</html>
```

### Branding Configuration

#### Custom Domain Integration

When using custom domains:
- Login URL becomes `https://auth.yourcompany.com`
- Fully branded experience
- No "auth0.com" visible to users

#### Per-Application Branding

Different branding per application:
```javascript
// Application-specific logo
await auth0.loginWithRedirect({
  // Application uses its configured branding
});
```

#### Per-Organization Branding

For B2B multi-tenant:
- Each organization can have its own logo and colors
- Configured in Organization settings
- Applied when user logs in with organization context

## Lock Widget

### What is Lock?

**Lock** is Auth0's pre-built, embeddable login widget:
- Drop-in UI component
- Handles all authentication scenarios
- Multiple connection support
- Mobile-responsive
- Customizable appearance

### Lock vs auth0.js vs auth0-spa-js

| Feature | Lock | auth0.js | auth0-spa-js |
|---------|------|----------|--------------|
| **UI Included** | ✅ Yes | ❌ No | ❌ No |
| **Use Case** | Quick implementation | Custom UI needed | SPAs (recommended) |
| **Embedded Login** | Yes | Yes | No (redirect only) |
| **Universal Login** | Yes | Yes | ✅ Yes |
| **SPA Optimized** | No | No | ✅ Yes |
| **PKCE Built-in** | No | No | ✅ Yes |
| **Recommendation** | Legacy/Classic | Custom UI | ✅ Modern SPAs |

### Lock Configuration

```javascript
var lock = new Auth0Lock(clientId, domain, {
  // Theme options
  theme: {
    logo: 'https://example.com/logo.png',
    primaryColor: '#0066CC',
    labeledSubmitButton: true
  },
  
  // Language
  languageDictionary: {
    title: 'Login to MyApp',
    emailInputPlaceholder: 'your@email.com',
    passwordInputPlaceholder: 'your password'
  },
  
  // Authentication options
  auth: {
    redirectUrl: 'https://myapp.com/callback',
    responseType: 'code',
    params: {
      scope: 'openid profile email'
    }
  },
  
  // Allowed connections
  allowedConnections: ['google-oauth2', 'Username-Password-Authentication'],
  
  // Other options
  allowSignUp: true,
  allowForgotPassword: true,
  rememberLastLogin: true,
  socialButtonStyle: 'big'
});

lock.show();
```

### Lock Events

```javascript
lock.on('authenticated', function(authResult) {
  console.log('Token:', authResult.accessToken);
});

lock.on('authorization_error', function(error) {
  console.log('Error:', error.error);
});

lock.on('show', function() {
  console.log('Lock is shown');
});

lock.on('hide', function() {
  console.log('Lock is hidden');
});
```

## Auth0 SDK Comparison

### auth0-spa-js (Recommended for SPAs)

**Best for**: Single Page Applications (React, Angular, Vue)

**Features**:
- ✅ PKCE built-in
- ✅ Token caching
- ✅ Silent authentication
- ✅ Automatic token renewal
- ✅ Redirect-based flow only (Universal Login)
- ✅ TypeScript support

**Installation**:
```bash
npm install @auth0/auth0-spa-js
```

**Usage**:
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

// After redirect
await auth0.handleRedirectCallback();

// Get token
const token = await auth0.getTokenSilently();

// Get user
const user = await auth0.getUser();

// Logout
await auth0.logout({ returnTo: window.location.origin });
```

### auth0-react (React SDK)

**Best for**: React applications

**Features**:
- Built on auth0-spa-js
- React hooks and context
- Component wrappers

**Usage**:
```jsx
import { Auth0Provider, useAuth0 } from '@auth0/auth0-react';

// Provider wrapper
<Auth0Provider
  domain="your-tenant.auth0.com"
  clientId="your-client-id"
  authorizationParams={{
    redirect_uri: window.location.origin
  }}
>
  <App />
</Auth0Provider>

// In components
function LoginButton() {
  const { loginWithRedirect, logout, isAuthenticated, user } = useAuth0();
  
  if (isAuthenticated) {
    return (
      <div>
        <p>Hello, {user.name}</p>
        <button onClick={() => logout()}>Logout</button>
      </div>
    );
  }
  
  return <button onClick={() => loginWithRedirect()}>Login</button>;
}
```

### auth0.js (Legacy)

**Best for**: Custom login UI (if absolutely needed)

**Features**:
- Low-level authentication
- Supports embedded login
- More control, more responsibility

**Usage**:
```javascript
import auth0 from 'auth0-js';

const webAuth = new auth0.WebAuth({
  domain: 'your-tenant.auth0.com',
  clientID: 'your-client-id',
  redirectUri: 'https://myapp.com/callback',
  responseType: 'code',
  scope: 'openid profile email'
});

// Redirect to login
webAuth.authorize();

// Parse callback
webAuth.parseHash((err, authResult) => {
  if (authResult && authResult.accessToken) {
    // Success
  }
});
```

### SDK Selection Guide

| Scenario | Recommended SDK |
|----------|-----------------|
| React SPA | auth0-react |
| Angular SPA | @auth0/auth0-angular |
| Vue.js SPA | auth0-spa-js (with Vue plugin) |
| Generic SPA | auth0-spa-js |
| Server-side web app | express-openid-connect (Node.js) |
| Mobile (iOS) | Auth0.swift |
| Mobile (Android) | Auth0.Android |
| Custom embedded UI | auth0.js (not recommended) |
| Classic Universal Login | Lock |

## Prompt Parameter

### What is the Prompt Parameter?

The `prompt` parameter controls the login page behavior:

```javascript
await auth0.loginWithRedirect({
  authorizationParams: {
    prompt: 'login' // Force login, even if session exists
  }
});
```

### Prompt Values

#### `prompt=login`
- **Forces re-authentication**
- Ignores existing SSO session
- User must enter credentials again
- Use for: Step-up authentication, sensitive operations

```javascript
await auth0.loginWithRedirect({
  authorizationParams: {
    prompt: 'login'
  }
});
```

#### `prompt=consent`
- **Forces consent screen**
- Even if user previously consented
- Shows scope permissions
- Use for: When scopes have changed

```javascript
await auth0.loginWithRedirect({
  authorizationParams: {
    prompt: 'consent'
  }
});
```

#### `prompt=none`
- **Silent authentication**
- No user interaction
- Fails if login required
- Use for: Token renewal, checking session

```javascript
try {
  const token = await auth0.getTokenSilently({
    authorizationParams: {
      prompt: 'none'
    }
  });
} catch (error) {
  if (error.error === 'login_required') {
    // User needs to log in
    await auth0.loginWithRedirect();
  }
}
```

#### `prompt=select_account`
- **Account selection**
- Shows account chooser if multiple accounts
- Use for: Multi-account scenarios

```javascript
await auth0.loginWithRedirect({
  authorizationParams: {
    prompt: 'select_account'
  }
});
```

### Combining Prompt Values

Can combine with space separator:
```javascript
await auth0.loginWithRedirect({
  authorizationParams: {
    prompt: 'login consent' // Force login AND consent
  }
});
```

## max_age Parameter

### What is max_age?

The `max_age` parameter specifies the **maximum time since user's last authentication**:

```javascript
await auth0.loginWithRedirect({
  authorizationParams: {
    max_age: 3600 // Require re-auth if last login > 1 hour ago
  }
});
```

### How It Works

1. User authenticated 2 hours ago
2. Request with `max_age=3600` (1 hour)
3. 2 hours > 1 hour → User must re-authenticate
4. Fresh `auth_time` claim in ID token

### Use Cases

#### Step-Up Authentication
```javascript
// Require fresh authentication for sensitive action
async function performSensitiveAction() {
  await auth0.loginWithRedirect({
    authorizationParams: {
      max_age: 300, // Must have authenticated within 5 minutes
      prompt: 'login'
    }
  });
}
```

#### Compliance Requirements
```javascript
// Healthcare app: re-auth every 15 minutes
const MAX_AUTH_AGE = 15 * 60; // 15 minutes

await auth0.loginWithRedirect({
  authorizationParams: {
    max_age: MAX_AUTH_AGE
  }
});
```

#### Check auth_time in Token
```javascript
// Verify authentication freshness
function isAuthFresh(idToken, maxAgeSeconds) {
  const authTime = idToken.auth_time;
  const now = Math.floor(Date.now() / 1000);
  return (now - authTime) <= maxAgeSeconds;
}
```

### max_age vs prompt=login

| Parameter | Behavior |
|-----------|----------|
| `max_age=0` | Similar to `prompt=login`, forces re-auth |
| `max_age=3600` | Re-auth only if last login > 1 hour ago |
| `prompt=login` | Always forces re-auth, ignores session |

## Login Hint and Screen Hint

### login_hint

Pre-fill the email/username field:

```javascript
await auth0.loginWithRedirect({
  authorizationParams: {
    login_hint: 'user@example.com'
  }
});
```

**Use Cases**:
- User coming from invite link
- Email already known from previous step
- Streamlined onboarding

### screen_hint

Direct user to specific screen:

```javascript
// Go directly to signup
await auth0.loginWithRedirect({
  authorizationParams: {
    screen_hint: 'signup'
  }
});
```

**Values**:
- `signup` - Show registration form
- `login` - Show login form (default)

## Universal Login Configuration Options

### Application-Level Settings

**Dashboard → Applications → [Your App] → Settings**

- **Allowed Callback URLs**: Where to redirect after login
- **Allowed Logout URLs**: Where to redirect after logout
- **Allowed Web Origins**: For CORS (SPAs)

### Tenant-Level Settings

**Dashboard → Settings → Advanced**

- **Login Session Management**: SSO timeouts
- **Enable Seamless SSO**: Cross-application SSO

### Connection Settings

**Dashboard → Authentication → [Connection Type]**

- **Enable for Applications**: Which apps can use this connection
- **Sync user profile**: Keep profile updated on each login

## Passwordless with Universal Login

### Email Link (Magic Link)

```javascript
// Trigger magic link
await auth0.loginWithRedirect({
  authorizationParams: {
    connection: 'email',
    send: 'link'
  }
});
```

### Email Code

```javascript
// Trigger email code
await auth0.loginWithRedirect({
  authorizationParams: {
    connection: 'email',
    send: 'code'
  }
});
```

### SMS Code

```javascript
// Trigger SMS code
await auth0.loginWithRedirect({
  authorizationParams: {
    connection: 'sms'
  }
});
```

## Social Connection Buttons

### Styling Options

**Dashboard → Branding → Universal Login → Social Buttons**

- **Icon only**: Small buttons with provider icon
- **Icon + Text**: Full buttons with "Continue with Google"

### Connection Order

Connections appear in the order configured in Dashboard.

### Hiding Connections

Show only specific connections:
```javascript
// In Lock configuration
allowedConnections: ['google-oauth2', 'facebook']
```

## Key Exam Takeaways

✅ **Universal Login**: Auth0-hosted login page, recommended approach  
✅ **New vs Classic**: New Universal Login recommended for all new projects  
✅ **Lock**: Pre-built UI widget, used with Classic Universal Login  
✅ **auth0-spa-js**: Recommended SDK for SPAs, has PKCE built-in  
✅ **auth0-react**: React-specific wrapper around auth0-spa-js  
✅ **auth0.js**: Legacy SDK for custom embedded login (not recommended)  
✅ **prompt=login**: Force re-authentication, ignore SSO session  
✅ **prompt=none**: Silent authentication, no user interaction  
✅ **prompt=consent**: Force consent screen display  
✅ **prompt=select_account**: Show account chooser  
✅ **max_age**: Maximum seconds since last authentication  
✅ **max_age=0**: Force re-authentication (similar to prompt=login)  
✅ **login_hint**: Pre-fill email field  
✅ **screen_hint=signup**: Go directly to signup page  
✅ **Custom domains**: Make login URL fully branded (login.yourcompany.com)  
✅ **Branding options**: Logo, colors, fonts via Dashboard  
✅ **Per-organization branding**: Different branding per B2B customer  
✅ **Silent auth with prompt=none**: Fails with login_required if session expired  
✅ **Universal Login benefits**: No cross-origin issues, all connections work, security updates automatic  
