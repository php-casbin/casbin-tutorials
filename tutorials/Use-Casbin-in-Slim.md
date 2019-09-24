# Use Casbin in Slim Framework 4

[Slim](http://slimframework.com) is a PHP micro framework that helps you quickly write simple yet powerful web applications and APIs.

[Casbin](https://github.com/php-casbin/php-casbin) can be used as an authorization middleware in Slim Framework.

### Authentication

First authentication, then authorization. 

Here we use [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication). 

[slim-basic-auth](https://github.com/tuupola/slim-basic-auth) provided a PSR-7 and PSR-15 Basic Auth Middleware, You can install it with composer: `composer require tuupola/slim-basic-auth`.

```php
$app->add(new HttpBasicAuthentication([
	'users' => [
	    'root' => 't00r',
	    'somebody' => 'passw0rd',
	],
	'before' => function ($request, $arguments) {
	    return $request->withAttribute('user', $arguments['user']);
	},
]));
```

### Casbin authorization middleware

This example implements an authorization middleware.

It first gets the currently authenticated `user`, the `uri` and `method` of current Request, and then uses `Casbin` to enforce.

```php

namespace App\Middleware;

use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Psr7\Response;
use Casbin\Enforcer;

class Authorization
{
    /**
     * Authorization middleware invokable class.
     *
     * @param ServerRequest  $request PSR-7 request
     * @param RequestHandler $handler PSR-15 request handler
     *
     * @return Response
     */
    public function __invoke(Request $request, RequestHandler $handler): Response
    {
        $e = new Enforcer('config/rbac_model.conf', 'config/policy.csv');

        $user = $request->getAttribute('user');
        $uri = $request->getUri();
        $action = $request->getMethod();

        if ($user && !$e->enforce($user, $uri->getPath(), $action)) {
            $response = new Response();
            $response->withStatus(403)->getBody()->write('Unauthorized.');

            return $response;
        }

        $response = $handler->handle($request);

        return $response;
    }
}

```

The contents of the `config/rbac_model.conf` file are as follows:

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
m = g(r.sub, p.sub) && keyMatch2(r.obj, p.obj) && r.act == p.act
```

The contents of the `config/policy.csv` file are as follows:

```
p, root, /, GET
p, root, /users, GET
p, root, /users/:id, GET
```

### Create routes

```php
$app->get('/', function (Request $request, Response $response) {
    $response->getBody()->write('Hello Casbin !');
    return $response;
});

$app->group('/users', function (Group $group) {
    $group->get('', ListUsersAction::class);
    $group->get('/{id}', ViewUserAction::class);
});
```

### Casbin skeleton application

The complete example is here : [Casbin skeleton application with Slim Framework 4](https://github.com/php-casbin/casbin-with-slim).

It makes setting up a new `Casbin` skeleton application with Slim Framework 4 quick and easy.
