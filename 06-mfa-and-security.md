# Multi-Factor Authentication (MFA)

## MFA Fundamentals

### What is MFA?

Multi-Factor Authentication requires users to provide **two or more verification factors** to gain access:
- Significantly reduces risk of account compromise
- Even if password is stolen, attacker needs second factor
- Industry best practice for security

### Authentication Factors (Recap)

1. **Knowledge** (something you know)
   - Password, PIN, security questions

2. **Possession** (something you have)
   - Mobile device, hardware token, smart card

3. **Inherence** (something you are)
   - Fingerprint, face recognition, iris scan

**Important**: True MFA requires factors from **different categories**
- ✅ Password + SMS = MFA (knowledge + possession)
- ❌ Password + Security Question = NOT MFA (both knowledge)

## MFA in Auth0

### MFA Factors Supported

#### 1. One-Time Password (OTP)
- **TOTP Apps**: Google Authenticator, Authy, Microsoft Authenticator
- Time-based 6-digit codes
- Works offline
- Most common MFA method

#### 2. SMS
- One-time code sent via text message
- Requires phone number
- SMS provider needed (Twilio, etc.)
- Less secure than TOTP (SIM swapping risk)

#### 3. Email
- One-time code sent via email
- Fallback option
- Less secure (email compromise)

#### 4. Push Notifications
- **Auth0 Guardian** app
- Approve/deny on mobile device
- Most user-friendly
- Real-time authentication

#### 5. Voice
- Automated phone call with code
- Alternative to SMS
- Good for accessibility

#### 6. WebAuthn/Security Keys
- **FIDO2** compliant
- Hardware keys (YubiKey, Titan)
- Platform authenticators (Touch ID, Face ID, Windows Hello)
- Most secure option
- Phishing-resistant

#### 7. Duo Security
- Third-party MFA provider
- Push notifications and other factors
- Enterprise-grade features

#### 8. Recovery Codes
- One-time backup codes
- Used when primary factor unavailable
- Generate set of codes during enrollment
- Each code single-use

### MFA Configuration

#### Dashboard Setup
**Security → Multi-factor Auth**

1. **Select Factors**
   - Enable desired MFA methods
   - Configure providers (SMS, email, etc.)

2. **Define MFA Policies**
   - **Always**: Require MFA for all users
   - **Adaptive**: Require based on conditions (via Actions)
   - **Never**: MFA disabled

3. **Enrollment Settings**
   - Force enrollment on first login
   - Allow users to skip (temporary)
   - Remember browser (reduce friction)

### MFA Policies

#### Always Require MFA
- All users must enroll and use MFA
- Enforced at tenant or application level
- Highest security

#### Adaptive/Conditional MFA
- MFA required based on risk factors
- Implemented via Auth0 Actions
- **Risk factors**:
  - User role (require for admins)
  - Login location (unusual country)
  - IP address (VPN detection)
  - Device (new device)
  - Time of day
  - Connection type

#### Never (MFA Optional)
- Users can voluntarily enroll
- Not recommended for production

### MFA Enrollment

#### First-Time Enrollment Flow
1. User logs in successfully
2. Prompted to enroll MFA
3. User selects factor (if multiple available)
4. Completes enrollment (scan QR, verify SMS, etc.)
5. May receive recovery codes
6. MFA active for future logins

#### Forcing Enrollment
- Configure in Actions: `api.multifactor.enable()`
- Cannot bypass enrollment
- May allow temporary skip (configurable)

#### Re-enrollment
- User loses device
- Admin resets MFA
- User can re-enroll on next login

### Auth0 Guardian (Detailed)

#### What is Auth0 Guardian?

- Auth0's native MFA application
- **Free** with Auth0
- iOS and Android apps
- Push notifications + TOTP

#### Features

**Push Notifications**
- One-tap approval
- Shows login location and device info
- Real-time authentication
- Best user experience

**Time-based OTP**
- 6-digit codes refresh every 30 seconds
- Works offline
- Fallback if push fails

**Enrollment**
- QR code scan during setup
- Pairs device with user account
- Multiple devices supported

#### Guardian Configuration

**Security → Multi-factor Auth → Push Notification via Auth0 Guardian**

- Enable/disable push
- Enable/disable OTP
- Customize Guardian app appearance (logo, name)
- Configure policies

#### Guardian SDK
- Customize enrollment flow in your app
- White-label Guardian functionality
- Available for iOS and Android

### MFA via Actions

Implement conditional/adaptive MFA using Actions:

#### Example: Require MFA for Admins
```javascript
exports.onExecutePostLogin = async (event, api) => {
  const roles = event.user.app_metadata?.roles || [];
  
  if (roles.includes('admin')) {
    api.multifactor.enable('any', {
      allowRememberBrowser: false
    });
  }
};
```

#### Example: Require MFA for High-Risk Login
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Geographic risk
  const highRiskCountries = ['XX', 'YY', 'ZZ'];
  const userCountry = event.request.geoip?.countryCode;
  
  if (highRiskCountries.includes(userCountry)) {
    api.multifactor.enable('any');
    return;
  }
  
  // New device detection
  const knownDevices = event.user.user_metadata?.devices || [];
  const currentDevice = event.request.user_agent;
  
  if (!knownDevices.includes(currentDevice)) {
    api.multifactor.enable('any');
  }
};
```

#### Example: Require Specific MFA Factor
```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Require WebAuthn (security key) for high-privilege operations
  const requiresHardwareKey = event.transaction?.acr_values?.includes('require-hardware-mfa');
  
  if (requiresHardwareKey) {
    api.multifactor.enable('webauthn');
  }
};
```

### MFA API Methods

#### `api.multifactor.enable(provider, options)`

**Providers**:
- `'any'` - User's enrolled factor
- `'duo'` - Duo Security
- `'google-authenticator'` - TOTP apps
- `'guardian'` - Auth0 Guardian
- `'webauthn-roaming'` - Security keys
- `'webauthn-platform'` - Platform authenticators
- `'sms'` - SMS codes
- `'email'` - Email codes
- `'otp'` - One-time password

**Options**:
- `allowRememberBrowser: boolean` - Allow "remember this device"

### Remember Browser Feature

#### How It Works
- User can choose "Trust this device for 30 days"
- Browser cookie stores trust
- MFA not required on trusted device
- Cookie expires after configured period

#### Configuration
- Set in MFA settings
- Duration configurable (default: 30 days)
- Can be disabled per-application
- Can be overridden in Actions: `allowRememberBrowser: false`

#### Security Considerations
- ⚠️ Reduces security on shared computers
- ✅ Good for user-owned devices
- ✅ Improves UX significantly
- Consider disabling for admin users

### MFA Reset

#### User-Initiated Reset
- Lost device
- Contact support
- Admin resets via Dashboard or Management API

#### Admin Reset
**Dashboard**: User Management → Select User → Reset MFA

**Management API**:
```javascript
DELETE /api/v2/users/{userId}/multifactor/{provider}
```

#### Self-Service Reset
- Can implement via custom UI
- Use Management API
- Require strong verification (email link, support ticket)

### Recovery Codes

#### Purpose
- Backup when primary MFA factor unavailable
- One-time use codes
- Generated during enrollment

#### Configuration
- Enable in MFA settings
- User downloads/saves codes
- Each code can be used once
- Generate new set after use

#### Best Practices
✅ Provide clear instructions for storage  
✅ Recommend printing or secure storage  
✅ Limit number of codes (typically 10)  
✅ Regenerate after multiple uses  
✅ Log when recovery code is used  

## MFA Best Practices

### Security
✅ **Enable MFA for all privileged accounts** (admins, developers)  
✅ **Prefer hardware keys** for highest security  
✅ **Avoid SMS** when possible (SIM swapping risk)  
✅ **Monitor MFA enrollment rates**  
✅ **Alert on MFA resets** (potential compromise)  
✅ **Use adaptive MFA** for better UX + security balance  

### User Experience
✅ **Allow multiple factors** for flexibility  
✅ **Provide clear enrollment instructions**  
✅ **Remember browser** for trusted devices  
✅ **Offer recovery options** (codes, support)  
✅ **Educate users** on MFA benefits  
✅ **Gradual rollout** for large user bases  

### Implementation
✅ **Test thoroughly** before enforcing  
✅ **Have support plan** for locked-out users  
✅ **Monitor MFA metrics** (enrollment, usage, failures)  
✅ **Document MFA policies** for users  
✅ **Plan for MFA reset process**  
✅ **Consider compliance requirements** (may mandate MFA)  

## Common MFA Scenarios

### Scenario 1: B2C Application with Optional MFA
- Enable multiple factors (TOTP, SMS, Guardian)
- Let users opt-in from profile settings
- Encourage with banners/prompts
- Don't force initially

### Scenario 2: B2B SaaS with Mandatory MFA
- Force enrollment on first login
- Allow TOTP and WebAuthn
- Disable "remember browser" for admins
- Provide recovery codes
- Admin can reset user MFA

### Scenario 3: Adaptive MFA for Risk-Based Authentication
- Use Actions to evaluate risk
- Require MFA for:
  - New devices
  - Unusual locations
  - High-value transactions
  - After password change
- Allow remember browser for low-risk scenarios

### Scenario 4: Enterprise with Hardware Keys
- Require WebAuthn for all employees
- Distribute YubiKeys or similar
- No fallback to SMS/OTP
- Recovery via IT helpdesk only

## Attack Protection and Security Features

### Bot Detection

#### What is Bot Detection?

Auth0's bot detection protects against automated attacks by identifying and blocking bot traffic:
- Uses **CAPTCHA** challenges
- Analyzes traffic patterns
- Protects authentication endpoints
- Prevents credential stuffing and brute force attacks

#### How It Works

**Detection Methods**:
- Traffic analysis and anomaly detection
- Browser fingerprinting
- Behavioral analysis
- Challenge-response tests (CAPTCHA)

**When Triggered**:
- Suspicious login patterns
- High velocity from single IP
- Known bot signatures
- Unusual user agent patterns

#### Configuration

**Dashboard → Security → Attack Protection → Bot Detection**

**Options**:
- **Disabled**: No bot detection (not recommended)
- **Enabled**: Show CAPTCHA for suspicious traffic
- **CAPTCHA Provider**: Google reCAPTCHA v2 or v3

**reCAPTCHA v2**:
- Traditional "I'm not a robot" checkbox
- Image challenges when uncertain
- More user friction

**reCAPTCHA v3**:
- Invisible, score-based detection
- No user interaction (unless very suspicious)
- Better user experience
- Analyzes user behavior

#### Best Practices

✅ Enable bot detection for all production tenants  
✅ Use reCAPTCHA v3 for better UX  
✅ Monitor CAPTCHA challenge rates  
✅ Combine with rate limiting  
✅ Test thoroughly (avoid blocking legitimate users)  

### Brute Force Protection

#### What is Brute Force Protection?

Protects against password guessing attacks by blocking IPs after repeated failed login attempts.

#### How It Works

**Triggers**:
- Multiple failed login attempts from same IP
- Failed attempts across multiple accounts
- Credential stuffing patterns

**Default Thresholds**:
- **10 failed attempts** from same IP for same user
- **100 failed attempts** from same IP across different users (within 24 hours)
- **10 attempts per second** per IP address

#### Configuration

**Dashboard → Security → Attack Protection → Brute Force Protection**

**Settings**:
- **Enabled/Disabled**: Toggle brute force protection
- **Shields**: Which triggers to activate
- **Allowlist**: IPs to exclude from blocking
- **Notification**: Email alerts on blocks

**Shields**:
1. **User ID + IP address**: Blocks IP after failed attempts for specific user
2. **IP address**: Blocks IP after failed attempts across any users

#### IP Address Blocking

**When an IP is Blocked**:
- All authentication attempts from that IP are rejected
- User sees "Too many attempts" error
- Applies to all users from that IP

**An IP address remains blocked until one of the following occurs**:

1. ✅ **Administrator removes the block**
   - Manual unblock via Dashboard
   - Or via Management API

2. ✅ **Administrator raises the Brute Force Threshold**
   - Increasing limit may auto-unblock
   - Not recommended as permanent solution

3. ✅ **The affected user changes their password on all linked accounts**
   - Password change triggers automatic unblock
   - Must change on ALL linked identities
   - Verifies legitimate user regained control

**Important**: Simply waiting does NOT unblock an IP automatically

#### Unblocking IPs

**Via Dashboard**:
1. Security → Attack Protection → Blocked IPs
2. Select IP address
3. Click "Unblock"

**Via Management API**:
```javascript
DELETE /api/v2/attack-protection/brute-force-protection/blocked-ips/{ip}
```

#### Allowlisting IPs

Add trusted IPs to bypass brute force protection:
- Corporate office IPs
- Testing environments
- Monitoring services
- Known good actors

**Configure**: Security → Attack Protection → Brute Force Protection → Allowlist

### Suspicious IP Throttling

#### What is Suspicious IP Throttling?

Rate limiting for IP addresses exhibiting suspicious behavior:
- Limits requests per second
- Protects against DDoS
- Prevents resource exhaustion

#### Rate Limits

**Default Throttling Rates**:
- **Per IP**: 10 requests per second per endpoint
- **Per User**: 10 requests per second per user ID
- **Global**: Varies by tenant plan

**When Throttled**:
- HTTP 429 (Too Many Requests) response
- Retry-After header indicates wait time
- Temporary blocking (not permanent like brute force)

#### Configuration

**Dashboard → Security → Attack Protection → Suspicious IP Throttling**

**Options**:
- Enable/disable throttling
- Configure thresholds (enterprise plans)
- Set allowlist for trusted IPs

#### Throttling vs Brute Force Protection

| Feature | Suspicious IP Throttling | Brute Force Protection |
|---------|--------------------------|------------------------|
| **Trigger** | High request rate | Failed login attempts |
| **Response** | Rate limit (429) | Block IP completely |
| **Duration** | Temporary (seconds/minutes) | Until manually unblocked or password changed |
| **Purpose** | Prevent DDoS, resource exhaustion | Prevent password guessing |
| **Threshold** | Requests per second | Failed attempts count |

### Breached Password Detection

#### What is Breached Password Detection?

Checks passwords against databases of known compromised credentials:
- Prevents use of leaked passwords
- Checks during signup and password change
- Integrates with Have I Been Pwned and other sources

#### Configuration

**Dashboard → Security → Attack Protection → Breached Password Detection**

**Options**:
- Enable at signup
- Enable at login
- Select action: Block or trigger Action for custom handling

#### Actions on Breach Detection

**Block**: Prevent signup/login with breached password

**Custom Action**: Handle via Auth0 Action for custom flow:
```javascript
exports.onExecutePostLogin = async (event, api) => {
  if (event.user.password_breached) {
    api.access.deny('Your password has been compromised. Please reset it.');
  }
};
```

## Tenant Logs

### What are Tenant Logs?

Auth0 tenant logs capture all authentication and authorization events:
- Security auditing and compliance
- Debugging authentication issues
- Monitoring user activity
- Detecting suspicious behavior

### Log Event Types

#### Authentication Events

| Event Code | Description |
|------------|-------------|
| **s** | Successful login |
| **ss** | Successful silent authentication |
| **f** | Failed login |
| **fu** | Failed login (invalid email/username) |
| **fp** | Failed login (wrong password) |
| **fc** | Failed by connector (AD/LDAP) |
| **fco** | Failed by CORS |
| **ssa** | Silent authentication successful |
| **fsa** | Silent authentication failed |
| **pwd_leak** | Password leaked (breached password) |

#### User Account Events

| Event Code | Description |
|------------|-------------|
| **sv** | Email verification successful |
| **fv** | Email verification failed |
| **sv** | Email verified |
| **scpr** | Change password request successful |
| **scp** | Change password successful |
| **fcp** | Change password failed |
| **du** | User deleted |
| **su** | User signup successful |
| **fs** | User signup failed |
| **pwd_reset** | Password reset requested |
| **ublkdu** | User unblocked by admin |

#### MFA Events

| Event Code | Description |
|------------|-------------|
| **gd_send_sms** | MFA SMS sent |
| **gd_start_enroll** | MFA enrollment started |
| **gd_enroll_success** | MFA enrollment successful |
| **gd_otp_rate_limit_exceed** | MFA OTP rate limit exceeded |
| **gd_auth_failed** | MFA authentication failed |
| **gd_auth_succeed** | MFA authentication successful |
| **gd_recovery_succeed** | MFA recovery code used successfully |
| **gd_unenroll** | MFA unenrollment |

#### Token Events

| Event Code | Description |
|------------|-------------|
| **sce** | Token exchange successful |
| **fce** | Token exchange failed |
| **seccft** | Client credentials exchange successful |
| **feccft** | Client credentials exchange failed |
| **seacft** | Authorization code exchange successful |
| **feacft** | Authorization code exchange failed |
| **sertft** | Refresh token exchange successful |
| **fertft** | Refresh token exchange failed |
| **slo** | Logout successful |

#### API Events

| Event Code | Description |
|------------|-------------|
| **sapi** | Management API successful |
| **fapi** | Management API failed |
| **scoa** | Consent accepted |
| **fcoa** | Consent failed |

### Log Data Structure

Each log entry contains:

```json
{
  "date": "2024-01-15T10:30:00.000Z",
  "type": "s",
  "description": "Successful login",
  "connection": "Username-Password-Authentication",
  "connection_id": "con_xxx",
  "client_id": "abc123",
  "client_name": "My App",
  "ip": "192.168.1.1",
  "user_agent": "Mozilla/5.0...",
  "details": {
    "completedAt": 1705317000000,
    "initiatedAt": 1705316900000
  },
  "user_id": "auth0|12345",
  "user_name": "user@example.com",
  "log_id": "90020231215104556xxx"
}
```

### Log Retention

| Tenant Plan | Retention Period |
|-------------|------------------|
| **Free** | 2 days |
| **Developer** | 2 days |
| **Developer Pro** | 10 days |
| **Enterprise** | 30 days |

**Note**: Logs are stored for audit purposes; for longer retention, export to external storage.

### Accessing Logs

#### Dashboard
**Monitoring → Logs**
- Search and filter logs
- View log details
- Export to CSV

#### Management API

**Get Logs**:
```javascript
const ManagementClient = require('auth0').ManagementClient;
const management = new ManagementClient({
  domain: '{tenant}.auth0.com',
  clientId: '{clientId}',
  clientSecret: '{clientSecret}'
});

// Get all logs
const logs = await management.logs.getAll({ per_page: 50 });

// Get specific log by ID
const log = await management.logs.get({ id: 'LOG_ID' });

// Filter by event type
const failedLogins = await management.logs.getAll({
  q: 'type:f*',  // All failed events
  per_page: 100
});
```

#### Log Search Syntax

**Query Examples**:
```
type:f                          # Failed events
type:s                          # Successful logins
user_id:"auth0|12345"           # Specific user
client_id:"abc123"              # Specific application
ip:"192.168.1.1"                # Specific IP
date:[2024-01-01 TO 2024-01-31] # Date range
type:f AND user_id:"auth0|123"  # Combined filters
```

### Log Export and Streaming

#### Log Streams (Real-Time Export)

Export logs in real-time to external services:

| Service | Use Case |
|---------|----------|
| **Amazon EventBridge** | AWS integration, triggers |
| **Azure Event Hub** | Azure integration |
| **Datadog** | Monitoring, alerting |
| **Splunk** | Security analytics |
| **Sumo Logic** | Log analysis |
| **Mixpanel** | Product analytics |
| **Segment** | Customer data platform |
| **Webhook** | Custom endpoints |

**Configuration**:
**Monitoring → Streams → Create Log Stream**

#### Extensions for Log Export

**Auth0 Log Extensions** (older method):
- Export to Amazon S3
- Export to Azure Blob Storage
- Export to Loggly
- Export to Papertrail

### Using Logs for Security

#### Detecting Suspicious Activity

**Action Example - Log and Alert on Suspicious Login**:
```javascript
exports.onExecutePostLogin = async (event, api) => {
  const riskIndicators = [];
  
  // Check for new device
  if (event.authentication.deviceId && 
      !event.user.app_metadata?.known_devices?.includes(event.authentication.deviceId)) {
    riskIndicators.push('new_device');
  }
  
  // Check for unusual location
  if (event.request.geoip?.country_code !== 
      event.user.app_metadata?.usual_country) {
    riskIndicators.push('unusual_location');
  }
  
  // Log security event with custom data
  if (riskIndicators.length > 0) {
    console.log(JSON.stringify({
      event: 'suspicious_login',
      user_id: event.user.user_id,
      indicators: riskIndicators,
      ip: event.request.ip,
      location: event.request.geoip?.country_name
    }));
    
    // Could also trigger MFA or notify admin
    api.multifactor.enable('any');
  }
};
```

#### Key Log Events to Monitor

| Event | Indicates | Action |
|-------|-----------|--------|
| `f`, `fp`, `fu` | Failed logins | Watch for patterns (brute force) |
| `pwd_leak` | Breached password | Force password reset |
| `gd_auth_failed` | Failed MFA | Possible attack |
| `limit_mu` | Too many users | Approaching plan limit |
| `fsa` | Silent auth failed | Session issues |
| `du` | User deleted | Audit if unexpected |

### Best Practices

✅ **Set up Log Streams** for real-time monitoring  
✅ **Configure Alerts** for security events (failed logins, breached passwords)  
✅ **Export to Long-Term Storage** for compliance (beyond retention period)  
✅ **Monitor Rate Limit Events** to detect attacks  
✅ **Track MFA Events** to ensure enrollment and usage  
✅ **Use Search Queries** effectively for troubleshooting  
✅ **Integrate with SIEM** (Splunk, Datadog) for enterprise security  

## Key Exam Takeaways

✅ **MFA requires 2+ factors from different categories** (knowledge, possession, inherence)  
✅ **Auth0 supports**: TOTP, SMS, Email, Push (Guardian), WebAuthn, Voice, Duo, Recovery Codes  
✅ **Auth0 Guardian**: Free native MFA app, push notifications + OTP  
✅ **MFA policies**: Always, Adaptive (via Actions), Never  
✅ **Adaptive MFA**: Use Actions with `api.multifactor.enable(provider, options)`  
✅ **WebAuthn/FIDO2**: Most secure, phishing-resistant, hardware keys or platform authenticators  
✅ **Remember browser**: Reduces MFA prompts for 30 days (configurable)  
✅ **Recovery codes**: Backup one-time codes for when primary factor unavailable  
✅ **MFA reset**: Admin via Dashboard or Management API, user loses MFA access  
✅ **SMS risk**: Vulnerable to SIM swapping, less secure than TOTP  
✅ **TOTP**: Works offline, time-based 6-digit codes  
✅ **Enrollment**: Can force via Actions, user completes setup on first login  
✅ **api.multifactor.enable()**: Trigger MFA in Actions, specify provider and options  
✅ **Bot Detection**: Uses CAPTCHA to block automated attacks  
✅ **Brute Force Protection**: Blocks IP after 10 failed attempts (same user) or 100 (different users)  
✅ **IP unblock conditions**: Admin removes block, threshold raised, OR user changes password on ALL linked accounts  
✅ **Throttling**: 10 requests/second per IP, returns HTTP 429  
✅ **Throttling vs Brute Force**: Throttling = rate limit (temporary), Brute Force = IP block (until unblocked)  
✅ **Breached Password Detection**: Checks against known compromised password databases  
✅ **Tenant Logs**: Capture all auth events - security, auditing, debugging  
✅ **Log codes**: s = success, f = failed, ss = silent auth success, fsa = silent auth failed  
✅ **Log retention**: Free/Developer = 2 days, Developer Pro = 10 days, Enterprise = 30 days  
✅ **Log Streams**: Real-time export to Datadog, Splunk, AWS EventBridge, Azure Event Hub, webhooks  
✅ **Management API logs**: `management.logs.getAll()` - query with Lucene syntax  
✅ **Log search syntax**: type:f (failed), user_id:"auth0|123", date:[2024-01-01 TO 2024-01-31]  
✅ **Key events to monitor**: f/fp/fu (failed logins), pwd_leak (breached), gd_auth_failed (MFA fail)  
