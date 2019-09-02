# Use with database adapter

In Casbin, the policy storage is implemented as an adapter (aka middleware for Casbin). A Casbin user can use an adapter to load policy rules from a storage (aka LoadPolicy()), or save policy rules to it (aka SavePolicy()).

The adapters should implement the `Casbin\Persist\Adapter` interface.

### Use Database adapter

Here we use the officially provided [Database adapter](https://github.com/php-casbin/database-adapter).

```
composer require casbin/database-adapter
```

### Usage

```
use Casbin\Enforcer;
use CasbinAdapter\Database\Adapter;

$config = [
    'type'     => 'mysql', // mysql,pgsql,sqlite,sqlsrv
    'hostname' => '127.0.0.1',
    'database' => 'test',
    'username' => 'root',
    'password' => '',
    'hostport' => '3306',
];

$adapter = Adapter::newAdapter($config);

$enforcer = new Enforcer('path/to/model.conf', $adapter);
```
