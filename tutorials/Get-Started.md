# Get Started

The three core concepts of Casbin are: `Model`, `Policy`, `Enforcer`.

In `Casbin`, an access control model is abstracted into a `CONF` file based on the PERM metamodel (Policy, Effect, Request, Matchers).

`Policy` is the stored dynamic policy rules, which can be stored in a `.csv` file or in a `database`.

`Enforcer` decides whether a "subject" can access a "object" with the operation "action".

### Installation

Via `composer`.

```
composer require casbin/casbin
```

### Try it

Create a `model.conf` and `policy.csv` file.

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

New a Casbin enforcer with a model file and a policy file:

```
use Casbin\Enforcer;

$enforcer = new Enforcer("path/to/model.conf", "path/to/policy.csv");
```

Add an enforcement hook into your code right before the access happens

```
$enforcer->enforce('alice', 'data1', 'read'); // true
$enforcer->enforce('alice', 'data2', 'write'); // false

$enforcer->enforce('bob', 'data1', 'read'); // false
$enforcer->enforce('bob', 'data2', 'write'); // true
```