# Route Middleware

[![Build status](https://img.shields.io/travis/phapi/middleware-route.svg?style=flat-square)](https://travis-ci.org/phapi/middleware-route)
[![Code Climate](https://img.shields.io/codeclimate/github/phapi/middleware-route.svg?style=flat-square)](https://codeclimate.com/github/phapi/middleware-route)
[![Test Coverage](https://img.shields.io/codeclimate/coverage/github/phapi/middleware-route.svg?style=flat-square)](https://codeclimate.com/github/phapi/middleware-route/coverage)

The Route Middleware package contains two middleware: <code>Phapi\Middleware\Route\Route</code> and <code>Phapi\Middleware\Route\Dispatch</code> that handles routing and dispatch to [endpoints](https://github.com/phapi/endpoint).

The Route Middleware will invoke the first route that matches the current HTTP request’s URI and method. If the request URI doesn't match any defined route a **404 Not Found** will be returned to the client. If the request method isn't implemented a **405 Method Not Allowed** will be returned instead.

The router does three things to be able to find a match, first it tries to find a direct match. This is possible for routes that doesn't contain regular expressions. If no direct match can be found the router will look in it's cache for a match. As a last resort the router tries to match against the routes that contains regular expressions. If a match is found it will be added to the cache.

## Installation
This middleware is by default included in the [Phapi Framework](https://github.com/phapi/phapi-framework) but if you need to install it it's available to install via [Packagist](https://packagist.org) and [Composer](https://getcomposer.org).

```shell
$ php composer.phar require phapi/middleware-route:1.*
```

## Configuration
The configuration contains three steps:

- Create middleware objects
- Define routes
- Add to pipeline

### Create middleware objects

```php
<?php

use Phapi\Middleware\Route\Route;
use Phapi\Middleware\Route\Router;
use Phapi\Middleware\Route\RouteParser;
use Phapi\Middleware\Route\Dispatch;

// Use the dependency injection container to get the configured cache
$route = new Route(new Router(new RouteParser, $container['cache']));
```

### Define routes
Routes can be added by passing an array (with the route as the key and the name of the resource as the value) to the **addRoutes** function.

```php
<?php

// Create a list of routes
$routes = [
  '/users'                  => '\\Phapi\\Endpoint\\Users',
  '/users/{name:a}'         => '\\Phapi\\Endpoint\\User',
  '/articles/{id:[0-9]+}'   => '\\Phapi\\Endpoint\\Article',
  '/blog/{slug}/{title:c}?' => '\\Phapi\\Endpoint\\Blog\\Post',
];

$route->addRoutes($routes);
```

By default a route pattern syntax is used where **{foo}** specified a placeholder with name **foo** and matching the string **[^/]+**. To adjust the pattern the placeholder matches, you can specify a custom pattern by writing **{bar:[0-9]+}**.

#### Regex Shortcuts
```php
:i => :/d+                # numbers only
:a => :[a-zA-Z0-9]+       # alphanumeric
:c => :[a-zA-Z0-9+_-\.]+  # alnumnumeric and +-_. characters
:h => :[a-fA-F0-9]+       # hex
```

#### Get params in an Endpoint
The router will inject the params in to the called method on the Endpoint. Lets use this route as an example:

```php
<?php
$routes = [
  '/users/{name:a}'         => '\\Phapi\\Endpoint\\User',
];
```

If we want to get the <code>name</code> value from <code>/users/phapi</code> we need to specify an endpoint looking like this:

```php
<?php

class User extends Endpoint {

  public function get($name) // $name will contain 'phapi'
  {
    ...
  }
}

```

### Add to pipeline
Last but not least, add the middleware to the pipeline. The <code>Dispatcher</code> doesn't need any configuration.

```php
<?php
$pipeline->pipe($route);
$pipeline->pipe(new \Phapi\Middleware\Route\Dispatcher());

```

### Complete example

```php
<?php

use Phapi\Middleware\Route\Route;
use Phapi\Middleware\Route\Router;
use Phapi\Middleware\Route\RouteParser;
use Phapi\Middleware\Route\Dispatch;

// Use the dependency injection container to get the configured cache
$route = new Route(new Router(new RouteParser, $container['cache']));

// Create a list of routes
$routes = [
  '/users'                  => '\\Phapi\\Endpoint\\Users',
  '/users/{name:a}'         => '\\Phapi\\Endpoint\\User',
  '/articles/{id:[0-9]+}'   => '\\Phapi\\Endpoint\\Article',
  '/blog/{slug}/{title:c}?' => '\\Phapi\\Endpoint\\Blog\\Post',
];

$route->addRoutes($routes);

$pipeline->pipe($route);
$pipeline->pipe(new \Phapi\Middleware\Route\Dispatcher());
```

See the [configuration documentation](http://phapi.github.io/docs/started/configuration/) for more information about how to configure the integration with the Phapi Framework.

## Phapi
This middleware is a Phapi package used by the [Phapi Framework](https://github.com/phapi/phapi-framework). The middleware are also [PSR-7](https://github.com/php-fig/http-message) compliant and implements the [Phapi Middleware Contract](https://github.com/phapi/contract).

## License
The Route Middleware is licensed under the MIT License - see the [license.md](https://github.com/phapi/middleware-route/blob/master/license.md) file for details

## Contribute
Contribution, bug fixes etc are [always welcome](https://github.com/phapi/middleware-route/issues/new).
