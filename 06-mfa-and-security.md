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
