# RBAC and Authorization

## Authorization Fundamentals

### Authentication vs Authorization (Recap)

- **Authentication**: Who are you? (Identity verification)
- **Authorization**: What can you do? (Permission/access control)

**Flow**: First authenticate (login) → Then authorize (check permissions)

## Role-Based Access Control (RBAC)

### What is RBAC?

- Access control based on **roles** assigned to users
- Role = collection of permissions
- Users inherit permissions from their roles
- Industry standard for authorization

### RBAC Components

#### 1. Users
- Individual people or service accounts
- Can have one or more roles
- Identified by unique ID (`user_id`)

#### 2. Roles
- Named collection of permissions
- Examples: Admin, Editor, Viewer, Manager
- Reusable across users
- Hierarchical (can inherit from other roles in some systems)

#### 3. Permissions
- Specific actions on resources
- Format: `action:resource` or `resource:action`
- Examples:
  - `read:articles`
  - `write:articles`
  - `delete:articles`
  - `manage:users`

#### 4. Resources
- What permissions apply to
- Usually your APIs or data objects
- Examples: articles, users, orders, reports

### RBAC Example

```
User: john@example.com
Roles: Editor

Role: Editor
Permissions:
- read:articles
- write:articles
- update:articles

Role: Admin
Permissions:
- read:articles
- write:articles
- update:articles
- delete:articles
- manage:users
```

## RBAC in Auth0

### Setting Up RBAC

Auth0 supports RBAC natively through APIs and Roles:

#### 1. Define Your API

**Dashboard → Applications → APIs → Create API**

- Name: Your API name
- Identifier: Unique URI (e.g., `https://api.myapp.com`)
- Signing Algorithm: RS256 (recommended)

#### 2. Create Permissions

In the API settings:
- **Permissions tab**
- Add permissions (scopes)
- Format: `action:resource`
- Examples:
  - `read:posts`
  - `write:posts`
  - `delete:posts`
  - `read:users`
  - `manage:settings`

#### 3. Create Roles

**User Management → Roles → Create Role**

- Role name: Admin, Editor, Viewer, etc.
- Description: What this role does
- Assign permissions to role

#### 4. Assign Roles to Users

**User Management → Users → Select User → Roles tab**

- Add one or more roles
- User inherits all permissions from assigned roles

### RBAC Settings

**API → Settings → RBAC Settings**

#### Enable RBAC
- ✅ Turn on to use roles and permissions
- Without this, scopes work like OAuth scopes (client-based)

#### Add Permissions in Access Token
- ✅ Include user permissions in access token
- Permissions appear in `permissions` claim
- API can check permissions without calling Auth0

#### Token Payload
```json
{
  "iss": "https://your-tenant.auth0.com/",
  "sub": "auth0|123456",
  "aud": "https://api.myapp.com",
  "permissions": [
    "read:posts",
    "write:posts",
    "delete:posts"
  ]
}
```

### Permissions in Access Tokens

#### How Permissions Get in Token

1. User has roles assigned
2. Roles have permissions
3. User logs in
4. Auth0 aggregates all permissions from all roles
5. Permissions included in access token (if enabled)

#### Checking Permissions

**In Your API**:
```javascript
// Extract token
const token = req.headers.authorization.split(' ')[1];

// Validate and decode JWT
const decoded = jwt.verify(token, publicKey);

// Check permissions
if (decoded.permissions.includes('delete:posts')) {
  // Allow deletion
} else {
  // Deny access
  res.status(403).json({ error: 'Insufficient permissions' });
}
```

### Dynamic Roles and Permissions

#### Using Actions to Assign Roles

You can dynamically assign roles based on conditions:

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Add role based on email domain
  if (event.user.email.endsWith('@admin.com')) {
    // Note: You'd typically use Management API to assign role
    // This example shows adding permission to token
    api.accessToken.setCustomClaim('https://myapp.com/roles', ['admin']);
  }
  
  // Add permissions to token
  const permissions = event.user.app_metadata?.permissions || [];
  if (permissions.length > 0) {
    api.accessToken.setCustomClaim('https://myapp.com/permissions', permissions);
  }
};
```

#### Using Management API

Programmatically manage roles:

```javascript
// Assign role to user
POST /api/v2/users/{user_id}/roles
{
  "roles": ["rol_abc123"]
}

// Get user's roles
GET /api/v2/users/{user_id}/roles

// Get user's permissions
GET /api/v2/users/{user_id}/permissions
```

## Authorization Strategies

### 1. RBAC (Role-Based Access Control)

**When to Use**:
- ✅ Clear role hierarchy
- ✅ Permissions grouped logically
- ✅ Users fit into defined roles
- ✅ Medium complexity

**Example**: CMS with Admin, Editor, Author, Viewer roles

### 2. ABAC (Attribute-Based Access Control)

**When to Use**:
- Complex authorization rules
- Context-dependent permissions
- Dynamic decision-making
- Fine-grained control

**Attributes**:
- User attributes (department, seniority)
- Resource attributes (owner, sensitivity)
- Environmental (time, location, device)

**Example**: 
- "Allow edit if user.department == document.department AND user.level >= document.minLevel AND time.hour < 18"

**Implementation in Auth0**:
- Use Actions to evaluate attributes
- Add context to tokens
- Make decision in API based on attributes

### 3. ACL (Access Control Lists)

**When to Use**:
- Per-resource permissions
- User-specific access
- Sharing/collaboration features

**Example**:
- Document sharing: User A can view Doc 1, User B can edit Doc 1
- Not practical for large scale (too many individual permissions)

**Implementation**:
- Store ACLs in your database
- Check in application logic
- Not typically managed in Auth0

### 4. Relationship-Based Access Control (ReBAC)

**When to Use**:
- Permissions based on relationships
- Social networks, collaboration platforms
- Graph-based models

**Example**:
- "Allow if user is member of same organization"
- "Allow if user is friend of resource owner"

**Implementation**:
- Model relationships in your database
- Check relationships in API
- Use Auth0 Organizations for org-based relationships

## Scopes vs Permissions

### Scopes (OAuth 2.0)

- **What**: Requested by client application
- **Purpose**: Limit what **application** can do
- **Granted to**: Access token for specific app
- **Example**: `read:email`, `openid`, `profile`

### Permissions (RBAC)

- **What**: Assigned to users via roles
- **Purpose**: Limit what **user** can do
- **Granted to**: Users based on their roles
- **Example**: `delete:posts`, `manage:users`

### Combination

Access token can contain both:
- **Scopes**: What app is allowed to request
- **Permissions**: What user is allowed to do

Final access = **intersection** of scopes and permissions:
- App requests scope `write:posts`
- User has permission `write:posts`
- ✅ Allow

- App requests scope `delete:posts`
- User has permission `write:posts` (not delete)
- ❌ Deny (user lacks permission)

## Authorization in Different Scenarios

### API Authorization

#### Protect API Endpoints

```javascript
// Middleware to check permissions
function requirePermission(permission) {
  return (req, res, next) => {
    const token = extractToken(req);
    const decoded = verifyToken(token);
    
    if (!decoded.permissions.includes(permission)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    next();
  };
}

// Use in routes
app.delete('/posts/:id', requirePermission('delete:posts'), (req, res) => {
  // Delete post
});
```

#### Check Multiple Permissions

```javascript
function requireAnyPermission(...permissions) {
  return (req, res, next) => {
    const token = extractToken(req);
    const decoded = verifyToken(token);
    
    const hasPermission = permissions.some(p => 
      decoded.permissions.includes(p)
    );
    
    if (!hasPermission) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    next();
  };
}

app.get('/admin', requireAnyPermission('admin:read', 'superadmin'), (req, res) => {
  // Admin page
});
```

### Frontend Authorization

#### Hide/Show UI Elements

```javascript
// React example
function DeleteButton({ postId }) {
  const { user } = useAuth0();
  const permissions = user['https://myapp.com/permissions'] || [];
  
  if (!permissions.includes('delete:posts')) {
    return null; // Hide button
  }
  
  return <button onClick={() => deletePost(postId)}>Delete</button>;
}
```

**Important**: Always enforce on backend, frontend checks are for UX only

### M2M Authorization

Machine-to-machine apps also use scopes/permissions:

1. Define API with permissions
2. Create M2M application
3. Authorize M2M app for API
4. Grant specific permissions to M2M app
5. M2M app gets token with granted permissions

```javascript
// M2M token includes granted permissions
{
  "iss": "https://tenant.auth0.com/",
  "sub": "abc123@clients",
  "aud": "https://api.myapp.com",
  "scope": "read:data write:data"
}
```

## Best Practices

### Role Design
✅ **Start simple** (few roles, expand as needed)  
✅ **Name roles clearly** (describe what they do)  
✅ **Avoid role explosion** (too many roles = maintenance burden)  
✅ **Use permission groups** (roles) not individual permissions  
✅ **Review roles periodically** (remove unused ones)  

### Permission Design
✅ **Granular but not excessive** (balance)  
✅ **Consistent naming** (`action:resource` format)  
✅ **Document permissions** (what they allow)  
✅ **Principle of least privilege** (minimal necessary access)  
✅ **Namespace permissions** (avoid conflicts)  

### Implementation
✅ **Enforce authorization in API** (never trust client)  
✅ **Validate tokens** (signature, claims, expiration)  
✅ **Check permissions on every request**  
✅ **Log authorization failures** (security monitoring)  
✅ **Cache permissions carefully** (invalidate when changed)  
✅ **Test authorization rules** (unit and integration tests)  

### Security
✅ **Enable RBAC in API settings**  
✅ **Add permissions to access tokens**  
✅ **Use namespaced custom claims** (`https://myapp.com/roles`)  
✅ **Audit role assignments** (who has admin?)  
✅ **Monitor permission usage** (detect anomalies)  
✅ **Regular access reviews** (remove stale permissions)  

## Common Patterns

### Admin vs User

```javascript
Roles:
- Admin (full access)
- User (limited access)

Permissions:
Admin:
- read:*
- write:*
- delete:*
- manage:users

User:
- read:own-data
- write:own-data
```

### Hierarchical Roles

```javascript
Roles (in order of privilege):
- Viewer (read only)
- Editor (view + edit)
- Manager (view + edit + delete)
- Admin (everything)
```

Implement by assigning cumulative permissions.

### Department-Based

```javascript
Roles:
- Sales-Manager
- Sales-Rep
- Engineering-Lead
- Engineer

Permissions include department context:
- read:sales-data
- write:engineering-docs
```

### Multi-Tenant

```javascript
Permissions include tenant context:
- read:org123:data
- write:org456:data

Or use Auth0 Organizations + generic permissions:
- read:data (scoped to user's organization automatically)
```

## Key Exam Takeaways

✅ **RBAC**: Users → Roles → Permissions → Resources  
✅ **Enable RBAC** in API settings to use roles and permissions  
✅ **Add permissions to access token** setting includes permissions in token  
✅ **Permissions format**: `action:resource` (e.g., `read:posts`, `delete:users`)  
✅ **Roles** group permissions, assigned to users  
✅ **Scopes vs Permissions**: Scopes = what app can do, Permissions = what user can do  
✅ **Final access** = intersection of scopes AND permissions  
✅ **Define permissions** in API settings in Auth0 Dashboard  
✅ **Assign roles** to users via Dashboard or Management API  
✅ **Check permissions** in API by validating access token claims  
✅ **Frontend checks** are UX only, always enforce on backend  
✅ **Custom claims** must be namespaced (e.g., `https://myapp.com/roles`)  
✅ **Management API** for programmatic role/permission management  
✅ **M2M apps** can also have scopes/permissions for API access  
✅ **Principle of least privilege**: Grant minimum necessary permissions  
