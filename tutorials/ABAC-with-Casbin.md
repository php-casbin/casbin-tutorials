# ABAC with Casbin

ABAC is Attribute-Based Access Control, meaning you can use the attributes (properties) of the subject, object or action instead of themselves (the string) to control the access.

Use the official `ABAC` example for example:

```
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == r.obj.owner
```

Here is a definition for the `r.obj` class:

```php
$data1 = new \stdClass();
$data1->name = 'data1';
$data1->owner = 'alice';

$data2 = new \stdClass();
$data2->name = 'data2';
$data2->owner = 'bob';
```

Then, use `Enforcer` to `enforce`:

```php
$e->enforce('alice', $data1, 'read');  // true
$e->enforce('alice', $data2, 'read');  // false

$e->enforce('bob', $data1, 'read');  // false
$e->enforce('bob', $data2, 'read');  // true
```