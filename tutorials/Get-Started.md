# Get Started

### Installation

```
composer require casbin/casbin
```

### Get started

#### Create a `model.conf` and `policy.csv` file.

`model.conf`:

```
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

`policy.csv`:

```
p, alice, data1, read
p, bob, data2, write
```

#### New a Casbin enforcer with a model file and a policy file:

```
use Casbin\Enforcer;

$enforcer = new Enforcer("path/to/model.conf", "path/to/policy.csv");
```

#### Add an enforcement hook into your code right before the access happens

```
$enforcer->enforce('alice', 'data1', 'read'); // true
$enforcer->enforce('alice', 'data2', 'write'); // false

$enforcer->enforce('bob', 'data1', 'read'); // false
$enforcer->enforce('bob', 'data2', 'write'); // true
```