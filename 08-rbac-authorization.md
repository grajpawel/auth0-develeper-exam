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

## RBAC Best Practices and Use Cases

### When to Use RBAC

#### ✅ RBAC is Ideal For:

**1. Clear Role Hierarchy**
- Organization has defined job roles
- Permissions naturally group by role
- Examples:
  - CMS: Admin, Editor, Author, Viewer
  - SaaS: Owner, Admin, Member, Guest
  - Healthcare: Doctor, Nurse, Receptionist, Patient

**2. Medium-Sized User Base**
- 10-10,000 users
- Manageable number of roles (3-20)
- Roles are relatively stable

**3. Standard Permission Sets**
- Most users fit into predefined roles
- Exceptions are rare
- Limited need for per-user customization

**4. Compliance Requirements**
- Need to demonstrate role-based access
- Audit requirements for who can do what
- Separation of duties needed

#### ❌ RBAC May Not Be Best For:

**1. Highly Dynamic Permissions**
- Permissions change frequently per user
- Context-dependent access (time, location, resource ownership)
- Better: ABAC (Attribute-Based Access Control)

**2. Very Granular Access**
- Per-resource ownership (my documents vs your documents)
- Relationship-based access (friends can see posts)
- Better: ACL or ReBAC

**3. Very Small Applications**
- 2-3 users
- Simple admin/user distinction
- Overhead of RBAC not needed

**4. Very Large Role Sets**
- Need 100+ different roles
- Role explosion problem
- Better: ABAC or hybrid approach

### RBAC Design Best Practices

#### 1. Role Naming and Organization

✅ **Use Business Language**
```javascript
Good:
- Customer Support Agent
- Sales Manager
- Product Owner

Avoid:
- ROLE_CSA_LVL2
- PERM_SALES_003
- USER_TYPE_B
```

✅ **Keep Role Count Manageable**
- Start with 3-5 core roles
- Add only when truly needed
- Aim for < 20 roles total
- If exceeding 20, reconsider design

✅ **Avoid Role Explosion**
```javascript
Bad (too many roles):
- Sales-Manager-East
- Sales-Manager-West
- Sales-Manager-North
- Sales-Manager-South
- Engineering-Manager-Frontend
- Engineering-Manager-Backend
// ... 100+ roles

Better:
- Sales-Manager
- Engineering-Manager
(Use attributes or metadata for region/team)
```

#### 2. Permission Design

✅ **Consistent Naming Convention**
```javascript
Standard format: action:resource

read:posts
write:posts
update:posts
delete:posts
publish:posts

read:users
create:users
update:users
delete:users
```

✅ **Granular but Not Excessive**
```javascript
Good balance:
- read:articles
- write:articles
- publish:articles
- delete:articles

Too granular (too many permissions):
- read:article:title
- read:article:body
- read:article:author
- read:article:tags
// Hard to manage

Too coarse:
- articles:all
// Not flexible enough
```

✅ **Use Wildcard Permissions Sparingly**
```javascript
// For admin/superuser only
*:*  // All permissions

// Department-level
read:sales:*
write:engineering:*

// Not recommended for regular users
```

#### 3. Role Assignment Strategy

✅ **Default Roles for New Users**
```javascript
exports.onExecutePostUserRegistration = async (event, api) => {
  // Assign default role to all new users
  const roleId = 'rol_viewer123'; // Viewer role
  
  // Would use Management API to assign
  // (Actions don't have direct role assignment)
  api.user.setAppMetadata('default_role', 'viewer');
};
```

✅ **Multiple Roles When Needed**
```javascript
User: alice@example.com
Roles:
  - Developer (access to code repos)
  - Reviewer (access to PR reviews)
  - On-Call (access to production systems)

// All permissions combined
```

✅ **Temporary Role Elevation**
```javascript
// On-call rotation
exports.onExecutePostLogin = async (event, api) => {
  const onCallSchedule = await getOnCallSchedule();
  
  if (onCallSchedule.currentUser === event.user.user_id) {
    // Add on-call permissions temporarily
    api.accessToken.setCustomClaim('https://myapp.com/on_call', true);
  }
};
```

#### 4. Permission Inheritance

✅ **Hierarchical Roles (Conceptually)**
```javascript
// Define role hierarchy in your app
Admin includes:
  - All Manager permissions
  - delete:users
  - manage:billing

Manager includes:
  - All Member permissions
  - approve:requests
  - assign:tasks

Member:
  - read:projects
  - create:tasks
  - update:own-tasks

// Implement by assigning cumulative permissions
```

❌ **Avoid Complex Inheritance Trees**
```javascript
Bad:
SuperAdmin → Admin → Manager → TeamLead → Member → Guest
// Too many levels, hard to understand

Better:
Admin → Member
Manager → Member
// Clear and simple
```

### RBAC Implementation Patterns

#### Pattern 1: Simple RBAC (Small Apps)

**Use Case**: Simple SaaS, internal tools
**Roles**: 3-5 roles (Admin, Member, Viewer)
**Implementation**: Direct role assignment in Auth0

```javascript
// In Auth0: Define roles and assign to users

// In API: Check permissions from token
if (!user.permissions.includes('delete:projects')) {
  return res.status(403).json({ error: 'Forbidden' });
}
```

**Pros**: Simple, easy to understand  
**Cons**: Not flexible for complex scenarios

#### Pattern 2: RBAC with Groups (Medium Apps)

**Use Case**: Departments, teams, divisions
**Roles**: Base roles + group membership
**Implementation**: RBAC + group metadata

```javascript
// User metadata
{
  "roles": ["member"],
  "groups": ["engineering", "security-team"]
}

// Action to add group-based permissions
exports.onExecutePostLogin = async (event, api) => {
  const groups = event.user.app_metadata?.groups || [];
  
  if (groups.includes('security-team')) {
    // Add security-specific permissions
    const securityPerms = ['read:security-logs', 'create:security-incidents'];
    api.accessToken.setCustomClaim('https://myapp.com/extra_perms', securityPerms);
  }
};
```

**Pros**: Flexible, handles department/team structure  
**Cons**: More complex to manage

#### Pattern 3: Dynamic RBAC (Large Apps)

**Use Case**: Large enterprises, complex organizations
**Roles**: Many roles + dynamic assignment
**Implementation**: RBAC + external authorization service

```javascript
// Call external authorization service
exports.onExecutePostLogin = async (event, api) => {
  // Get dynamic permissions from authorization service
  const response = await axios.post('https://authz.company.com/permissions', {
    userId: event.user.user_id,
    context: {
      department: event.user.app_metadata.department,
      location: event.request.geoip.country_code,
      time: new Date().toISOString()
    }
  });
  
  // Add dynamic permissions to token
  api.accessToken.setCustomClaim('https://myapp.com/permissions', response.data.permissions);
};
```

**Pros**: Very flexible, handles complex rules  
**Cons**: More complex infrastructure, external dependency

### RBAC Anti-Patterns to Avoid

#### ❌ Anti-Pattern 1: Role Per User

```javascript
Bad:
- role_john_smith
- role_jane_doe
- role_bob_jones

// This defeats the purpose of RBAC
// Use individual permissions or ACLs instead
```

#### ❌ Anti-Pattern 2: Permissions in Role Names

```javascript
Bad:
- ReadArticlesWriteArticles
- DeleteUsersManageSettings

Better:
- Editor (has read:articles, write:articles)
- Admin (has delete:users, manage:settings)
```

#### ❌ Anti-Pattern 3: No Default Deny

```javascript
Bad:
// Check if user has permission
if (user.permissions.includes('read:data')) {
  return data;
}
// No else - returns undefined, possible security hole

Good:
if (user.permissions.includes('read:data')) {
  return data;
} else {
  throw new Error('Forbidden');
}
```

#### ❌ Anti-Pattern 4: Client-Side Only Authorization

```javascript
Bad:
// Only check permissions in frontend
if (user.hasPermission('delete:post')) {
  <button onClick={deletePost}>Delete</button>
}

// API doesn't check - anyone can call DELETE endpoint

Good:
// Frontend check for UX
// API ALWAYS validates permissions
app.delete('/posts/:id', requirePermission('delete:posts'), deletePost);
```

#### ❌ Anti-Pattern 5: Hardcoding Permissions

```javascript
Bad:
if (user.role === 'admin') {
  // Allow access
}

Better:
if (user.permissions.includes('manage:users')) {
  // Allow access
}

// Allows flexibility to give permission to non-admins
```

### RBAC Governance

#### Regular Access Reviews

✅ **Quarterly Reviews**
- Review who has which roles
- Remove unnecessary access
- Update roles as org changes

✅ **Automated Alerts**
```javascript
// Alert on high-privilege role assignment
exports.onExecutePostLogin = async (event, api) => {
  const roles = event.authorization?.roles || [];
  
  if (roles.includes('admin')) {
    await sendAlert({
      type: 'high_privilege_login',
      user: event.user.email,
      role: 'admin',
      ip: event.request.ip
    });
  }
};
```

#### Audit Logging

✅ **Log All Authorization Decisions**
```javascript
function checkPermission(user, permission, resource) {
  const hasPermission = user.permissions.includes(permission);
  
  // Log the decision
  auditLog.log({
    timestamp: new Date(),
    user: user.id,
    permission: permission,
    resource: resource,
    granted: hasPermission,
    ip: user.ip
  });
  
  return hasPermission;
}
```

✅ **Track Role Changes**
- Who assigned the role
- When it was assigned
- Why (if possible)
- When it was removed

#### Principle of Least Privilege

✅ **Start with Minimum Access**
```javascript
// New user gets minimal role
Default role: Viewer

// Request additional access as needed
User requests: Editor role
Manager approves: ✓
System assigns: Editor role
Audit log: timestamp, approver, reason
```

✅ **Time-Limited Elevated Access**
```javascript
exports.onExecutePostLogin = async (event, api) => {
  const tempAdmin = event.user.app_metadata?.temp_admin;
  
  if (tempAdmin && tempAdmin.expires > Date.now()) {
    // Grant temporary admin access
    api.accessToken.setCustomClaim('https://myapp.com/temp_admin', true);
  }
};
```

### RBAC Performance Considerations

#### ✅ Cache Role/Permission Lookups

```javascript
// Cache user permissions
const permissionCache = new Map();

function getUserPermissions(userId) {
  if (permissionCache.has(userId)) {
    return permissionCache.get(userId);
  }
  
  const permissions = fetchPermissionsFromDB(userId);
  permissionCache.set(userId, permissions);
  
  // Invalidate cache after 5 minutes
  setTimeout(() => permissionCache.delete(userId), 5 * 60 * 1000);
  
  return permissions;
}
```

#### ✅ Include Permissions in Token

```javascript
// Enable "Add Permissions to Access Token" in API settings
// Permissions included in token, no DB lookup needed

// Token payload
{
  "permissions": [
    "read:posts",
    "write:posts",
    "delete:posts"
  ]
}
```

#### ⚠️ Watch Token Size

```javascript
// Too many permissions can make token too large
// Limit permissions per role
// Use permission groups if needed

// If token too large, consider:
1. Reducing number of permissions
2. Using permission groups
3. Fetching permissions server-side (not in token)
```

### RBAC Testing Strategies

#### ✅ Test Each Role

```javascript
describe('Article Permissions', () => {
  test('Viewer can read articles', async () => {
    const viewer = createUserWithRole('viewer');
    const response = await viewer.get('/articles');
    expect(response.status).toBe(200);
  });
  
  test('Viewer cannot delete articles', async () => {
    const viewer = createUserWithRole('viewer');
    const response = await viewer.delete('/articles/1');
    expect(response.status).toBe(403);
  });
  
  test('Admin can delete articles', async () => {
    const admin = createUserWithRole('admin');
    const response = await admin.delete('/articles/1');
    expect(response.status).toBe(200);
  });
});
```

#### ✅ Test Permission Combinations

```javascript
test('User with multiple roles gets combined permissions', () => {
  const user = {
    roles: ['developer', 'reviewer'],
    permissions: ['read:code', 'write:code', 'approve:pr']
  };
  
  expect(hasPermission(user, 'read:code')).toBe(true);
  expect(hasPermission(user, 'approve:pr')).toBe(true);
});
```

#### ✅ Test Edge Cases

```javascript
test('User with no roles has no permissions', () => {
  const user = { roles: [], permissions: [] };
  expect(hasPermission(user, 'read:anything')).toBe(false);
});

test('Deleted role removes permissions', async () => {
  await removeRoleFromUser(userId, 'admin');
  const user = await getUser(userId);
  expect(user.permissions).not.toContain('delete:users');
});
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
✅ **RBAC best for**: Clear role hierarchy, 3-20 roles, standard permission sets  
✅ **RBAC not ideal for**: Highly dynamic permissions, per-resource ownership, role explosion  
✅ **Role naming**: Use business language, keep count manageable (< 20)  
✅ **Permission naming**: Consistent format (action:resource), granular but not excessive  
✅ **Avoid role explosion**: Don't create role per user or per minor variation  
✅ **Default deny**: Always explicitly deny if permission missing  
✅ **Never rely on client-side**: Always validate permissions in API  
✅ **Cache permissions**: Improve performance, invalidate on changes  
✅ **Include permissions in token**: Avoid DB lookups, watch token size  
✅ **Regular access reviews**: Quarterly role reviews, remove stale access  
✅ **Audit logging**: Log all authorization decisions and role changes  
