# Auth0 Management API

## What is the Management API?

The Auth0 Management API allows you to **programmatically manage your Auth0 tenant**:
- Create, read, update, delete users
- Manage roles and permissions
- Configure connections
- Access logs
- Manage applications
- And much more

### Management API vs Authentication API

| Feature | Management API | Authentication API |
|---------|---------------|-------------------|
| **Purpose** | Tenant administration | User authentication |
| **Who Uses** | Your backend services | Your applications/users |
| **Authentication** | M2M access token | User credentials/tokens |
| **Examples** | Create users, assign roles | Login, get user info |
| **Base URL** | `https://{tenant}.auth0.com/api/v2` | `https://{tenant}.auth0.com` |

## Getting a Management API Token

### Method 1: M2M Application (Recommended)

Create a Machine-to-Machine application to get Management API access:

#### Step 1: Create M2M Application
**Dashboard → Applications → Create Application → Machine to Machine**

#### Step 2: Authorize for Management API
Select **Auth0 Management API** and grant required scopes.

#### Step 3: Get Token
```javascript
POST https://{tenant}.auth0.com/oauth/token
Content-Type: application/json

{
  "client_id": "your-m2m-client-id",
  "client_secret": "your-m2m-client-secret",
  "audience": "https://{tenant}.auth0.com/api/v2/",
  "grant_type": "client_credentials"
}

// Response
{
  "access_token": "eyJhbGciOiJS...",
  "token_type": "Bearer",
  "expires_in": 86400
}
```

#### Step 4: Use Token
```javascript
GET https://{tenant}.auth0.com/api/v2/users
Authorization: Bearer {access_token}
```

### Method 2: Test Token (Dashboard)

For testing only:
**Dashboard → Applications → APIs → Auth0 Management API → API Explorer**

Click "Create & Authorize Test Application" to get a test token.

⚠️ **Never use test tokens in production**

## Management API Scopes

### Scope Categories

Scopes control what operations are allowed. Format: `{action}:{resource}`

#### User Scopes
| Scope | Description |
|-------|-------------|
| `read:users` | Read user profiles |
| `create:users` | Create new users |
| `update:users` | Update user profiles |
| `delete:users` | Delete users |
| `read:user_idp_tokens` | Read IdP tokens |

#### Role Scopes
| Scope | Description |
|-------|-------------|
| `read:roles` | Read roles |
| `create:roles` | Create roles |
| `update:roles` | Update roles |
| `delete:roles` | Delete roles |

#### Connection Scopes
| Scope | Description |
|-------|-------------|
| `read:connections` | Read connections |
| `create:connections` | Create connections |
| `update:connections` | Update connections |
| `delete:connections` | Delete connections |

#### Client (Application) Scopes
| Scope | Description |
|-------|-------------|
| `read:clients` | Read applications |
| `create:clients` | Create applications |
| `update:clients` | Update applications |
| `delete:clients` | Delete applications |

#### Log Scopes
| Scope | Description |
|-------|-------------|
| `read:logs` | Read tenant logs |
| `read:logs_users` | Read user-specific logs |

#### Organization Scopes
| Scope | Description |
|-------|-------------|
| `read:organizations` | Read organizations |
| `create:organizations` | Create organizations |
| `update:organizations` | Update organizations |
| `delete:organizations` | Delete organizations |
| `read:organization_members` | Read org members |
| `create:organization_members` | Add org members |
| `delete:organization_members` | Remove org members |

### Principle of Least Privilege

✅ **Only grant scopes you need**
```javascript
// Good: Specific scopes
scopes: ['read:users', 'update:users']

// Bad: All scopes (security risk)
scopes: ['read:users', 'create:users', 'update:users', 'delete:users', ...]
```

## Common Management API Endpoints

### User Management

#### Get All Users
```javascript
GET /api/v2/users?
  page=0&
  per_page=50&
  include_totals=true&
  search_engine=v3&
  q=email:"*@example.com"

// Response
{
  "users": [
    {
      "user_id": "auth0|123456",
      "email": "user@example.com",
      "name": "John Doe",
      "picture": "https://...",
      "identities": [...],
      "app_metadata": {...},
      "user_metadata": {...}
    }
  ],
  "start": 0,
  "limit": 50,
  "total": 1
}
```

#### Get User by ID
```javascript
GET /api/v2/users/{user_id}

// Example
GET /api/v2/users/auth0%7C123456
// Note: user_id must be URL encoded (| becomes %7C)
```

#### Create User
```javascript
POST /api/v2/users
{
  "connection": "Username-Password-Authentication",
  "email": "newuser@example.com",
  "password": "SecurePassword123!",
  "name": "New User",
  "email_verified": false,
  "app_metadata": {
    "plan": "free",
    "signup_source": "api"
  },
  "user_metadata": {
    "preferences": {
      "language": "en"
    }
  }
}
```

#### Update User
```javascript
PATCH /api/v2/users/{user_id}
{
  "name": "Updated Name",
  "app_metadata": {
    "plan": "premium"
  },
  "blocked": false
}
```

#### Delete User
```javascript
DELETE /api/v2/users/{user_id}
```

#### Search Users
```javascript
// Search by email
GET /api/v2/users?q=email:"user@example.com"&search_engine=v3

// Search by name (wildcard)
GET /api/v2/users?q=name:*john*&search_engine=v3

// Search by app_metadata
GET /api/v2/users?q=app_metadata.plan:"premium"&search_engine=v3

// Combine with AND
GET /api/v2/users?q=email_verified:true AND app_metadata.plan:"premium"&search_engine=v3
```

### Role Management

#### Get All Roles
```javascript
GET /api/v2/roles

// Response
[
  {
    "id": "rol_abc123",
    "name": "Admin",
    "description": "Administrator role"
  },
  {
    "id": "rol_def456",
    "name": "User",
    "description": "Standard user role"
  }
]
```

#### Create Role
```javascript
POST /api/v2/roles
{
  "name": "Editor",
  "description": "Can edit content"
}
```

#### Assign Role to User
```javascript
POST /api/v2/users/{user_id}/roles
{
  "roles": ["rol_abc123", "rol_def456"]
}
```

#### Get User's Roles
```javascript
GET /api/v2/users/{user_id}/roles

// Response
[
  {
    "id": "rol_abc123",
    "name": "Admin",
    "description": "Administrator role"
  }
]
```

#### Remove Roles from User
```javascript
DELETE /api/v2/users/{user_id}/roles
{
  "roles": ["rol_abc123"]
}
```

#### Assign Permissions to Role
```javascript
POST /api/v2/roles/{role_id}/permissions
{
  "permissions": [
    {
      "resource_server_identifier": "https://api.myapp.com",
      "permission_name": "read:posts"
    },
    {
      "resource_server_identifier": "https://api.myapp.com",
      "permission_name": "write:posts"
    }
  ]
}
```

### Organization Management

#### Create Organization
```javascript
POST /api/v2/organizations
{
  "name": "acme-corp",
  "display_name": "Acme Corporation",
  "branding": {
    "logo_url": "https://acme.com/logo.png",
    "colors": {
      "primary": "#0066CC"
    }
  },
  "metadata": {
    "plan": "enterprise"
  }
}
```

#### Add Members to Organization
```javascript
POST /api/v2/organizations/{org_id}/members
{
  "members": ["auth0|user123", "auth0|user456"]
}
```

#### Assign Roles to Organization Member
```javascript
POST /api/v2/organizations/{org_id}/members/{user_id}/roles
{
  "roles": ["rol_orgadmin123"]
}
```

#### Create Organization Invitation
```javascript
POST /api/v2/organizations/{org_id}/invitations
{
  "inviter": {
    "name": "Admin User"
  },
  "invitee": {
    "email": "newmember@example.com"
  },
  "client_id": "your_client_id",
  "roles": ["rol_member123"],
  "ttl_sec": 604800
}
```

### Log Management

#### Get Logs
```javascript
GET /api/v2/logs?
  page=0&
  per_page=50&
  sort=date:-1&
  include_totals=true

// Response
{
  "logs": [
    {
      "log_id": "90020210101000000...",
      "date": "2024-01-15T10:30:00.000Z",
      "type": "s",
      "description": "Successful Login",
      "user_id": "auth0|123456",
      "user_name": "user@example.com",
      "ip": "192.168.1.1",
      "user_agent": "Mozilla/5.0..."
    }
  ],
  "total": 1000
}
```

#### Filter Logs by Type
```javascript
// Successful logins only
GET /api/v2/logs?q=type:s

// Failed logins only  
GET /api/v2/logs?q=type:f

// For specific user
GET /api/v2/logs?q=user_id:"auth0|123456"
```

### Connection Management

#### Get Connections
```javascript
GET /api/v2/connections

// Response
[
  {
    "id": "con_abc123",
    "name": "Username-Password-Authentication",
    "strategy": "auth0",
    "enabled_clients": ["client_id_1", "client_id_2"]
  }
]
```

#### Update Connection
```javascript
PATCH /api/v2/connections/{connection_id}
{
  "options": {
    "requires_username": false,
    "password_policy": "good"
  }
}
```

### MFA Management

#### Get User's MFA Enrollments
```javascript
GET /api/v2/users/{user_id}/enrollments

// Response
[
  {
    "id": "dev_abc123",
    "type": "totp",
    "name": "Google Authenticator",
    "created_at": "2024-01-15T10:00:00.000Z"
  }
]
```

#### Delete MFA Enrollment (Reset MFA)
```javascript
DELETE /api/v2/users/{user_id}/multifactor/{provider}

// Example: Reset TOTP
DELETE /api/v2/users/auth0%7C123456/multifactor/google-authenticator
```

#### Invalidate Remember Browser
```javascript
POST /api/v2/users/{user_id}/multifactor/actions/invalidate-remember-browser
```

## Rate Limits

### Understanding Rate Limits

The Management API has rate limits to prevent abuse:

| Endpoint Category | Rate Limit |
|------------------|------------|
| **User exports** | 5 requests/second |
| **User search** | 20 requests/second |
| **User CRUD** | 15 requests/second |
| **Roles** | 10 requests/second |
| **Logs** | 10 requests/second |
| **General** | Varies by plan |

### Rate Limit Headers

```http
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 7
X-RateLimit-Reset: 1642500000
```

### Handling Rate Limits

```javascript
async function callManagementAPI(url, options, retries = 3) {
  try {
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      // Rate limited
      const retryAfter = response.headers.get('Retry-After') || 1;
      
      if (retries > 0) {
        await sleep(retryAfter * 1000);
        return callManagementAPI(url, options, retries - 1);
      }
      
      throw new Error('Rate limit exceeded');
    }
    
    return response.json();
  } catch (error) {
    throw error;
  }
}
```

### Best Practices for Rate Limits

✅ **Cache responses** when possible  
✅ **Batch operations** (add multiple members at once)  
✅ **Use webhooks** instead of polling  
✅ **Implement exponential backoff**  
✅ **Monitor rate limit headers**  

## Pagination

### Standard Pagination

Most list endpoints support pagination:

```javascript
// First page
GET /api/v2/users?page=0&per_page=50&include_totals=true

// Response
{
  "users": [...],
  "start": 0,
  "limit": 50,
  "total": 500
}

// Next page
GET /api/v2/users?page=1&per_page=50
```

### Checkpoint Pagination (Logs)

For logs, use checkpoint pagination for reliable results:

```javascript
// First request
GET /api/v2/logs?take=100

// Use 'next' from response for subsequent requests
GET /api/v2/logs?take=100&from={last_log_id}
```

## Using Management API from Actions

### Accessing Management API in Actions

```javascript
const ManagementClient = require('auth0').ManagementClient;

exports.onExecutePostLogin = async (event, api) => {
  // Create Management API client
  const management = new ManagementClient({
    domain: event.secrets.AUTH0_DOMAIN,
    clientId: event.secrets.M2M_CLIENT_ID,
    clientSecret: event.secrets.M2M_CLIENT_SECRET,
    scope: 'read:users update:users'
  });
  
  // Update user metadata
  await management.updateUser(
    { id: event.user.user_id },
    {
      app_metadata: {
        last_login: new Date().toISOString()
      }
    }
  );
  
  // Assign role
  await management.assignRolestoUser(
    { id: event.user.user_id },
    { roles: ['rol_default123'] }
  );
};
```

### Important Considerations

⚠️ **Action Timeout**: 20 seconds max - Management API calls count against this  
⚠️ **Rate Limits**: Actions share rate limits with other API calls  
⚠️ **Secrets**: Store credentials in Action secrets  

## Management API SDKs

### Node.js SDK

```javascript
const { ManagementClient } = require('auth0');

const management = new ManagementClient({
  domain: 'your-tenant.auth0.com',
  clientId: 'your-client-id',
  clientSecret: 'your-client-secret'
});

// Get users
const users = await management.getUsers();

// Create user
const newUser = await management.createUser({
  connection: 'Username-Password-Authentication',
  email: 'user@example.com',
  password: 'Password123!'
});

// Update user
await management.updateUser(
  { id: 'auth0|123456' },
  { name: 'New Name' }
);
```

### Python SDK

```python
from auth0.management import Auth0

auth0 = Auth0(
    domain='your-tenant.auth0.com',
    token='your-access-token'
)

# Get users
users = auth0.users.list()

# Create user
new_user = auth0.users.create({
    'connection': 'Username-Password-Authentication',
    'email': 'user@example.com',
    'password': 'Password123!'
})
```

## Error Handling

### Common Error Codes

| Status Code | Meaning | Common Causes |
|-------------|---------|---------------|
| 400 | Bad Request | Invalid parameters |
| 401 | Unauthorized | Invalid/expired token |
| 403 | Forbidden | Missing required scope |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Server Error | Auth0 internal error |

### Error Response Format

```javascript
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Payload validation error: 'email' is required",
  "errorCode": "invalid_body"
}
```

### Error Handling Example

```javascript
try {
  const user = await management.createUser({
    connection: 'Username-Password-Authentication',
    email: 'user@example.com',
    password: 'weak'
  });
} catch (error) {
  if (error.statusCode === 400) {
    console.log('Validation error:', error.message);
  } else if (error.statusCode === 409) {
    console.log('User already exists');
  } else if (error.statusCode === 429) {
    console.log('Rate limited, retry later');
  } else {
    console.log('Unknown error:', error);
  }
}
```

## Security Best Practices

### Token Management

✅ **Short token lifetime**: Default 24 hours, reduce if possible  
✅ **Secure storage**: Never store in frontend code  
✅ **Rotate secrets**: Regularly rotate M2M client secrets  
✅ **Environment variables**: Store credentials securely  

### Scope Management

✅ **Least privilege**: Only grant necessary scopes  
✅ **Separate M2M apps**: Different apps for different purposes  
✅ **Audit scope usage**: Review what each app needs  

### Network Security

✅ **IP allowlisting**: Restrict API access by IP (enterprise)  
✅ **TLS only**: All API calls over HTTPS  
✅ **Monitor logs**: Watch for suspicious API usage  

## Key Exam Takeaways

✅ **Management API**: Programmatic tenant administration  
✅ **Authentication**: Requires M2M access token with correct scopes  
✅ **Token endpoint**: POST to `/oauth/token` with `client_credentials` grant  
✅ **Audience**: Must be `https://{tenant}.auth0.com/api/v2/`  
✅ **Scopes**: Format is `{action}:{resource}` (e.g., `read:users`, `update:roles`)  
✅ **Rate limits**: Vary by endpoint, implement retry with backoff  
✅ **User search**: Use `q` parameter with Lucene syntax  
✅ **Pagination**: Use `page`, `per_page`, `include_totals`  
✅ **User ID encoding**: Must URL encode `|` character (`%7C`)  
✅ **Role assignment**: POST to `/api/v2/users/{id}/roles`  
✅ **MFA reset**: DELETE `/api/v2/users/{id}/multifactor/{provider}`  
✅ **Log types**: `s` = success, `f` = failure, and many more  
✅ **Organization members**: POST to `/api/v2/organizations/{org_id}/members`  
✅ **Actions + Management API**: Use auth0 SDK, watch for timeout  
✅ **Error 429**: Rate limit exceeded, check Retry-After header  
✅ **Error 403**: Missing required scope for operation  
