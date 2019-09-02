# RBAC with Casbin

Here we use the officially provided [Database adapter](https://github.com/php-casbin/database-adapter).

### Use RBAC Model

Our model.conf has the following contents:

```
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && keyMatch2(r.obj, p.obj) && regexMatch(r.act, p.act)
```

### New a Casbin enforcer with RBAC Model

```php
use Casbin\Enforcer;

$enforcer = new Enforcer('path/to/model.conf', $adapter);
```

### Add Policy

#### Assign roles to Alice and bob 

```php
// alice has the admin role
$enforcer->addRoleForUser('alice', 'admin'); 
// bob has the member role
$enforcer->addRoleForUser('bob', 'member');
```

#### Assign permissions to roles

`member` has all read operations:

```php
$enforcer->addPermissionForUser('member', '/foo', 'GET');
$enforcer->addPermissionForUser('member', '/foo/:id', 'GET');
```

`admin` has all `read` and `wirte` operations:

```php
// admin inherits all permissions of member
$enforcer->addRoleForUser('admin', 'member');

$enforcer->addPermissionForUser('admin', '/foo', 'POST');
$enforcer->addPermissionForUser('admin', '/foo/:id', 'PUT');
$enforcer->addPermissionForUser('admin', '/foo/:id', 'DELETE');
```

#### Verify permissions

`alice` has all the permissions of `/foo`.

```php
$enforcer->enforce('alice', '/foo', 'GET'); // true
$enforcer->enforce('alice', '/foo', 'GET'); // true

$enforcer->enforce('alice', '/foo', 'POST'); // true
$enforcer->enforce('alice', '/foo/1', 'PUT'); // true
$enforcer->enforce('alice', '/foo/1', 'DELETE'); // true
```

`bob` has member role, only read permission for `/foo`.

```php
$enforcer->enforce('bob', '/foo', 'GET'); // true
$enforcer->enforce('bob', '/foo', 'GET'); // true

$enforcer->enforce('bob', '/foo', 'POST'); // false
$enforcer->enforce('bob', '/foo/1', 'PUT'); // false
$enforcer->enforce('bob', '/foo/1', 'DELETE'); // false
```