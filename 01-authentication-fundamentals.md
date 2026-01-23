# Authentication Fundamentals

## Authentication vs Authorization

### Authentication
- **What**: Verifying who a user is (identity verification)
- **Question**: "Who are you?"
- **Example**: Login with username/password, biometrics, SSO

### Authorization
- **What**: Determining what a user can access (permissions)
- **Question**: "What are you allowed to do?"
- **Example**: Role-based access control, permissions, scopes

## Authentication Factors

Authentication factors are categories of credentials used to verify identity:

### 1. Knowledge Factors (Something You Know)
- Passwords
- PINs
- Security questions
- Passphrases

### 2. Possession Factors (Something You Have)
- Mobile device (SMS, authenticator app)
- Hardware tokens
- Smart cards
- Email access

### 3. Inherence Factors (Something You Are)
- Biometrics:
  - Fingerprint
  - Facial recognition
  - Iris/retina scan
  - Voice recognition

### Multi-Factor Authentication (MFA)
- Combines **two or more** different factors
- Significantly increases security
- Common combinations:
  - Password + SMS code
  - Password + authenticator app (TOTP)
  - Password + biometric

## Login Credentials

### Traditional Credentials
- **Username/Email + Password**
  - Most common form
  - Stored securely (hashed and salted)
  - Subject to attacks (brute force, credential stuffing)

### Modern Approaches
- **Passwordless Authentication**
  - Email magic links
  - SMS one-time codes
  - Authenticator apps (push notifications)
  - WebAuthn/FIDO2

### Best Practices
- Strong password policies (length, complexity)
- Password breach detection
- Account lockout after failed attempts
- Secure password reset flows

## Authenticators

Authenticators are tools/methods used to prove identity. They align with the three authentication factor types:

### Knowledge-Based Authenticators (Something You Know)
- **Passwords/Passphrases**
  - Traditional text-based secrets
  - Stored securely (hashed and salted)
  - Weakest factor when used alone
  
- **PINs**
  - Numeric passwords
  - Often shorter than passwords
  - Common for device unlock

- **Security Questions**
  - Personal information verification
  - Easily guessable or researched
  - Not recommended as sole factor

### Possession-Based Authenticators (Something You Have)

#### Time-based One-Time Password (TOTP)
- Apps like Google Authenticator, Authy, Microsoft Authenticator
- Generates time-sensitive codes (usually 6 digits)
- Based on shared secret + current time
- No internet connection required
- **Possession factor**: Requires access to device with TOTP app

#### Push Notifications
- Auth0 Guardian
- Duo Push
- Receive authentication request on mobile device
- Approve/deny with single tap
- **Possession factor**: Requires access to registered device

#### SMS/Email Codes
- One-time codes sent via SMS or email
- Less secure than TOTP (SIM swapping, email compromise)
- Good for account recovery
- **Possession factor**: Requires access to phone or email

#### Hardware Tokens
- YubiKey, other FIDO2 devices
- Physical USB/NFC devices
- Most secure possession factor
- Phishing-resistant
- **Possession factor**: Physical device must be present

### Inherence-Based Authenticators (Something You Are)

#### Biometric Authenticators
- **Fingerprint** (Touch ID, ultrasonic sensors)
  - Fast and convenient
  - Unique to individual
  - Can't be changed if compromised
  
- **Facial Recognition** (Face ID, Windows Hello)
  - Contactless authentication
  - 3D mapping for security
  - Liveness detection to prevent photos
  
- **Iris/Retina Scan**
  - Extremely unique patterns
  - High accuracy
  - Less common in consumer devices
  
- **Voice Recognition**
  - Analyzes voice patterns
  - Can be affected by illness, environment
  - Often combined with other factors

#### Behavioral Biometrics
- Typing patterns (keystroke dynamics)
- Mouse movement patterns
- Gait analysis (how you walk)
- Used for continuous authentication

**Important**: Biometric authenticators are often combined with device possession factor (e.g., Touch ID on your iPhone = inherence + possession)

## Auth0 Guardian

Auth0's native MFA solution:

### Features
- **Push Notifications**: Approve/deny login requests
- **OTP Support**: Time-based one-time passwords
- **Mobile App**: iOS and Android applications
- **Easy Integration**: Built into Auth0 platform

### How It Works
1. User enrolls device during first login
2. QR code or SMS for setup
3. Guardian app generates TOTP codes
4. Can send push notifications for frictionless auth

### Configuration
- Enable in Auth0 Dashboard → Security → Multi-factor Auth
- Choose factors: Push, SMS, OTP, Email
- Set policies: Always, Adaptive, Never
- Customize Guardian app appearance

## FIDO (Fast Identity Online)

### What is FIDO?
- Industry standard for passwordless authentication
- Eliminates passwords using cryptographic keys
- Phishing-resistant authentication

### FIDO2 / WebAuthn
- **WebAuthn**: Browser API for authentication
- **CTAP**: Communication protocol for authenticators
- Supported by major browsers and platforms

### How It Works
1. **Registration**:
   - User registers device/authenticator
   - Public/private key pair generated
   - Public key stored on server
   - Private key stays on device (never shared)

2. **Authentication**:
   - Server sends challenge
   - User authenticates locally (PIN, biometric)
   - Device signs challenge with private key
   - Server verifies with public key

### Benefits
- **Phishing-Resistant**: Keys bound to domain
- **No Shared Secrets**: Private key never leaves device
- **User-Friendly**: Biometric or simple PIN
- **Privacy**: No PII transmitted

### Auth0 Support
- Auth0 supports WebAuthn as MFA factor
- Compatible with:
  - Security keys (YubiKey, Titan)
  - Platform authenticators (Touch ID, Face ID, Windows Hello)
  - Biometric devices

### Implementation Considerations
- Requires HTTPS
- Browser/device compatibility
- Fallback methods for unsupported devices
- User education and onboarding

## Authentication Assurance Levels (AAL)

### What is AAL?

**Authentication Assurance Level** measures the strength and confidence of the authentication process. Defined by NIST (National Institute of Standards and Technology) in Special Publication 800-63B.

### AAL Levels

#### AAL1 - Single-Factor Authentication
- **Requirement**: One authentication factor
- **Examples**:
  - Password only
  - Email magic link
  - SMS code alone
  
- **Use Cases**:
  - Low-risk applications
  - Public content access
  - Non-sensitive data
  
- **Security Level**: Basic

#### AAL2 - Multi-Factor Authentication
- **Requirement**: Two or more different authentication factors
- **Examples**:
  - Password + TOTP
  - Password + SMS
  - Password + Push notification
  - Password + Hardware token
  
- **Use Cases**:
  - Most modern applications
  - Access to personal data
  - Financial transactions
  - Healthcare records
  
- **Security Level**: Moderate to High
- **NIST Requirement**: Must use cryptographic mechanisms

#### AAL3 - Hardware-Based MFA
- **Requirement**: Hardware-based authentication with proof of possession
- **Examples**:
  - Password + Hardware security key (FIDO2/WebAuthn)
  - Password + Smart card
  - Cryptographic authenticator (YubiKey, Titan Key)
  
- **Use Cases**:
  - High-security environments
  - Government systems
  - Critical infrastructure
  - Privileged access management
  - Financial institutions
  
- **Security Level**: Very High
- **NIST Requirement**: Hardware-based cryptographic authenticator, resistant to impersonation

### AAL in Auth0

#### Requesting Specific AAL
You can request a specific AAL using the `acr_values` parameter:

```javascript
await auth0.loginWithRedirect({
  acr_values: 'http://schemas.openid.net/pape/policies/2007/06/multi-factor'
});
```

#### Enforcing AAL via Actions
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Require AAL2 for admin users
  const roles = event.user.app_metadata?.roles || [];
  
  if (roles.includes('admin')) {
    // Require multi-factor authentication (AAL2)
    api.multifactor.enable('any');
  }
  
  // Require AAL3 for highly privileged operations
  const requiresAAL3 = event.transaction?.acr_values?.includes('aal3');
  if (requiresAAL3) {
    // Require hardware-based MFA
    api.multifactor.enable('webauthn-roaming');
  }
};
```

#### AAL in Tokens
The achieved AAL can be included in ID tokens:
```json
{
  "sub": "auth0|123456",
  "amr": ["pwd", "otp"], // Authentication Methods References
  "acr": "aal2" // Authentication Context Class Reference
}
```

### AAL Comparison Table

| Level | Factors Required | Examples | Security | Use Case |
|-------|------------------|----------|----------|----------|
| **AAL1** | 1 factor | Password, Magic link | Basic | Low-risk apps |
| **AAL2** | 2+ factors | Password + TOTP, Password + SMS | Moderate-High | Most apps |
| **AAL3** | 2+ with hardware | Password + FIDO2 key | Very High | High-security |

## Step-Up Authentication vs Adaptive MFA

### Step-Up Authentication

#### What is Step-Up Authentication?

**Step-up authentication** requires additional authentication when accessing sensitive resources or performing high-risk actions, even if the user is already logged in.

#### How It Works
1. User logs in with standard authentication (e.g., password only)
2. User attempts to access sensitive resource or action
3. System requires additional authentication factor
4. User provides additional factor (e.g., TOTP, biometric)
5. User granted temporary elevated access

#### Use Cases
- **Financial transactions**: Transfer money requires re-authentication
- **Changing sensitive settings**: Email, password, security settings
- **Accessing privileged data**: Medical records, financial reports
- **Administrative actions**: Deleting accounts, changing permissions
- **Compliance requirements**: Regulations require re-authentication for certain actions

#### Example Scenario
```
User logs in with password (AAL1)
  ↓
User browses application normally
  ↓
User initiates wire transfer
  ↓
System requires step-up: "Please verify with authenticator app"
  ↓
User enters TOTP code (now AAL2)
  ↓
Transaction proceeds
  ↓
Elevated access expires after 15 minutes
```

#### Implementation in Auth0

**Using acr_values**:
```javascript
// Check if user needs step-up
if (requiresHighSecurity) {
  await auth0.loginWithRedirect({
    acr_values: 'http://schemas.openid.net/pape/policies/2007/06/multi-factor',
    max_age: 0 // Force fresh authentication
  });
}
```

**Using Actions**:
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Check for step-up request in transaction
  const requiresStepUp = event.transaction?.step_up;
  
  if (requiresStepUp) {
    // Force MFA even if recently authenticated
    api.multifactor.enable('any', {
      allowRememberBrowser: false
    });
  }
  
  // Add elevated privileges claim with expiration
  if (event.authentication?.methods?.length > 1) {
    api.accessToken.setCustomClaim('https://myapp.com/elevated', {
      granted_at: Date.now(),
      expires_in: 900 // 15 minutes
    });
  }
};
```

**In Your Application**:
```javascript
// Detect need for step-up
function requiresStepUp(action) {
  const sensitiveActions = ['transfer_money', 'change_email', 'delete_account'];
  return sensitiveActions.includes(action);
}

// Check if user has elevated access
function hasElevatedAccess(token) {
  const elevated = token['https://myapp.com/elevated'];
  if (!elevated) return false;
  
  const expiresAt = elevated.granted_at + (elevated.expires_in * 1000);
  return Date.now() < expiresAt;
}

// Request step-up if needed
if (requiresStepUp('transfer_money') && !hasElevatedAccess(accessToken)) {
  // Trigger re-authentication with MFA
  await performStepUp();
}
```

### Adaptive MFA

#### What is Adaptive MFA?

**Adaptive MFA** (also called **Risk-Based Authentication**) dynamically determines whether to require MFA based on risk assessment of the login attempt.

#### Risk Factors Evaluated
- **Location**: Unusual country, city, or IP address
- **Device**: New device or browser
- **Time**: Login at unusual time (3 AM when user normally logs in at 9 AM)
- **Velocity**: Multiple login attempts in short time
- **Network**: Public WiFi, VPN, Tor exit node
- **Behavior**: Anomalous user behavior patterns
- **Threat Intelligence**: IP on blocklist, known malicious actor

#### How It Works
1. User attempts to login
2. System calculates risk score based on context
3. **Low Risk**: Allow login without MFA (password only)
4. **Medium Risk**: Require MFA
5. **High Risk**: Block login or require additional verification

#### Example Scenarios

**Low Risk (No MFA)**:
- Same device used for past 30 days
- Same location (home city)
- Same time of day (work hours)
- Known IP address
- → Allow login with password only

**Medium Risk (Require MFA)**:
- New device
- OR different country
- OR unusual time
- → Require MFA

**High Risk (Block or Strong MFA)**:
- Multiple failed login attempts
- IP on threat list
- Impossible travel (Tokyo 1 hour ago, now New York)
- → Block or require hardware MFA (AAL3)

#### Implementation in Auth0

**Using Actions for Adaptive MFA**:
```javascript
exports.onExecutePostLogin = async (event, api) => {
  let riskScore = 0;
  
  // Check location
  const knownCountries = event.user.user_metadata?.knownCountries || [];
  const currentCountry = event.request.geoip?.countryCode;
  if (!knownCountries.includes(currentCountry)) {
    riskScore += 30; // New country
  }
  
  // Check device
  const knownDevices = event.user.user_metadata?.knownDevices || [];
  const currentDevice = event.request.user_agent;
  if (!knownDevices.includes(currentDevice)) {
    riskScore += 25; // New device
  }
  
  // Check time of day
  const hour = new Date().getHours();
  if (hour < 6 || hour > 22) {
    riskScore += 15; // Unusual time
  }
  
  // Check for high-risk IP
  const suspiciousIPs = await fetchSuspiciousIPs();
  if (suspiciousIPs.includes(event.request.ip)) {
    riskScore += 50; // Known bad IP
  }
  
  // Check impossible travel
  const lastLogin = event.user.last_login;
  const lastLocation = event.user.user_metadata?.lastLocation;
  if (isImpossibleTravel(lastLogin, lastLocation, currentCountry)) {
    riskScore += 100; // Impossible travel
  }
  
  // Apply adaptive policy
  if (riskScore >= 100) {
    // High risk: Block login
    api.access.deny('Suspicious activity detected. Please contact support.');
  } else if (riskScore >= 40) {
    // Medium risk: Require MFA
    api.multifactor.enable('any');
  }
  // Low risk (score < 40): Allow without MFA
  
  // Update user metadata with current login info
  api.user.setUserMetadata('lastLocation', currentCountry);
  api.user.setUserMetadata('knownDevices', 
    [...new Set([...knownDevices, currentDevice])].slice(-5) // Keep last 5 devices
  );
  api.user.setUserMetadata('knownCountries',
    [...new Set([...knownCountries, currentCountry])]
  );
};
```

**Using Third-Party Risk Signals**:
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Call external risk assessment API
  const riskAssessment = await axios.post('https://risk-api.example.com/assess', {
    userId: event.user.user_id,
    ip: event.request.ip,
    userAgent: event.request.user_agent,
    location: event.request.geoip
  });
  
  if (riskAssessment.data.riskLevel === 'high') {
    api.multifactor.enable('webauthn-roaming'); // Require hardware key
  } else if (riskAssessment.data.riskLevel === 'medium') {
    api.multifactor.enable('any'); // Require any MFA
  }
  // Low risk: no MFA required
};
```

### Step-Up Authentication vs Adaptive MFA Comparison

| Aspect | Step-Up Authentication | Adaptive MFA |
|--------|------------------------|--------------|
| **When Triggered** | When accessing sensitive resource | During initial login |
| **Decision Based On** | Resource sensitivity | Login risk factors |
| **User Already Logged In?** | Yes (requires re-auth) | No (part of initial auth) |
| **Frequency** | As needed per action | Every login attempt |
| **Purpose** | Elevate privileges temporarily | Reduce friction while maintaining security |
| **Example** | "Enter MFA to transfer money" | "New device detected, enter MFA" |
| **AAL Change** | AAL1 → AAL2/AAL3 temporarily | AAL1 or AAL2 based on risk |

### Best Practices

#### Step-Up Authentication
✅ Clear user communication about why additional auth is needed  
✅ Limited duration for elevated access (15-30 minutes)  
✅ Log all step-up events for audit  
✅ Allow multiple methods for step-up  
✅ Graceful degradation if user can't complete step-up  

#### Adaptive MFA
✅ Balance security with user experience  
✅ Use multiple risk signals (not just one)  
✅ Learn from user patterns over time  
✅ Provide feedback to users ("New device detected")  
✅ Allow users to mark devices as trusted  
✅ Monitor false positives (legitimate users blocked)  
✅ Implement gradual rollout  

### Combined Approach

Many organizations use **both**:
1. **Adaptive MFA** at login (risk-based)
2. **Step-Up Authentication** for sensitive actions (resource-based)

**Example Flow**:
```
Login attempt from known device + location
  ↓
Adaptive MFA: Low risk → Allow with password only (AAL1)
  ↓
User browses normally
  ↓
User attempts to change email address
  ↓
Step-Up: Require MFA for this action (AAL2)
  ↓
User provides TOTP
  ↓
Email change allowed
```

## Key Exam Takeaways

✅ **Know the three authentication factors**: Knowledge, Possession, Inherence  
✅ **Authenticators align with factors**: TOTP/hardware tokens = possession, biometrics = inherence, passwords = knowledge  
✅ **MFA requires 2+ different factor types**: Password + SMS is MFA  
✅ **AAL1**: Single factor (password only)  
✅ **AAL2**: Multi-factor (password + TOTP)  
✅ **AAL3**: Hardware-based MFA (password + FIDO2 key)  
✅ **Step-Up Authentication**: Require additional auth for sensitive actions, user already logged in  
✅ **Adaptive MFA**: Risk-based decision at login time, based on context (location, device, etc.)  
✅ **TOTP vs Push**: TOTP works offline, Push requires connectivity  
✅ **FIDO2/WebAuthn**: Phishing-resistant, passwordless, uses public-key cryptography  
✅ **Auth0 Guardian**: Built-in MFA with push notifications and OTP  
✅ **Biometric authenticators**: Often inherence + possession (device required)  
✅ **Risk factors for adaptive MFA**: Location, device, time, velocity, network, threat intelligence  
✅ **Step-up vs Adaptive**: Step-up = action-triggered, Adaptive = login-time risk assessment  
