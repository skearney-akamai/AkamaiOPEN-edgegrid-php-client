# akamai-open/edgegrid-client

[Akamai {OPEN} EdgeGrid Authentication](https://techdocs.akamai.com/developer/docs/set-up-authentication-credentials) for PHP

This library implements the Akamai {OPEN} EdgeGrid Authentication scheme on top of [Guzzle](https://github.com/guzzle/guzzle), as both a drop-in replacement client, and middleware.

For more information visit the [Akamai {OPEN} Developer Community](https://developer.akamai.com).

## Installation

This library requires PHP 8+ or HHVM 3.5+.

To install, use [`composer`](http://getcomposer.org):

```sh
$ composer require akamai-open/edgegrid-client
```

#### Alternative install methods

#### Single File (PHAR)

Download the PHAR file from the [releases](https://github.com/akamai/AkamaiOPEN-edgegrid-php/releases) page and include it inside your code:

    ```php
    include 'akamai-open-edgegrid-auth.phar';

    // Library is ready to use
    ```

## Use

The `Akamai\Open\EdgeGrid\Client` extends `\GuzzleHttp\Client` as a drop-in replacement to transparently sign API requests without changing other ways you use Guzzle.

To use the client, you call `\Akamai\Open\EdgeGrid\Client->setAuth()` or provide an instance of `\Akamai\Open\EdgeGrid\Authentication` to the constructor prior to making a request to an API.

```php
$client = new Akamai\Open\EdgeGrid\Client([
  'base_uri' => 'https://akaa-baseurl-xxxxxxxxxxx-xxxxxxxxxxxxx.luna.akamaiapis.net'
]);

$client->setAuth($client_token, $client_secret, $access_token);

// use $client just as you would \Guzzle\Http\Client
$response = $client->get('identity-management/v3/user-profile');
```

### Authentication

To generate your credentials, see [Create authentication credentials](https://techdocs.akamai.com/developer/docs/set-up-authentication-credentials).

We recommend using a `.edgerc` authentication file. Place your credentials under a heading of `[default]` and place it at the root/home directory of the web server user or in your current working directory.

```
[default]
client_secret = C113nt53KR3TN6N90yVuAgICxIRwsObLi0E67/N8eRN=
host = akab-h05tnam3wl42son7nktnlnnx-kbob3i3v.luna.akamaiapis.net
access_token = akab-acc35t0k3nodujqunph3w7hzp7-gtm6ij
client_token = akab-c113ntt0k3n4qtari252bfxxbsl-yvsdj
```

Call your `.edgerc` file one of two ways:

*  Use the factory method `\Akamai\Open\EdgeGrid\Client::createFromEdgeRcFile()`.

    ```php
    $client = \Akamai\Open\EdgeGrid\Client::createFromEdgeRcFile();

    // use $client just as you would \Guzzle\Http\Client
    $response = $client->get('/identity-management/v3/user-profile');
    ```

* Specify a credentials section and/or `.edgerc` location:

    ```php
    $client = \Akamai\Open\EdgeGrid\Client::createFromEdgeRcFile('example', '../config/.edgerc');

    // use $client just as you would \Guzzle\Http\Client
    $response = $client->get('/identity-management/v3/user-profile');
    ```

## Command Line Interface

This library provides a command line interface (CLI) with a limited set of capabilities that mimic [httpie](http://httpie.org).

### Install

Install the CLI with composer `vendor/bin/http` or execute the PHAR file.

```sh
# Composer installed
$ ./vendor/bin/http --help

# For Windows
> php ./vendor/bin/http --help

# PHAR download
php akamai-open-edgegrid-client.phar --help
```

### Arguments

You can set authentication, specify an HTTP method (case insensitive), headers, and JSON body fields.

> **Note:** Our CLI shows all HTTP and body data. JSON is formated.

* `--auth-type={edgegrid,basic,digest}` — Set the authentication type (default: none)
* `--auth user:` or `--a user:` — Set the `.edgerc` section to use. Unlike `httpie-edgegrid` the `:` is optional
* `Header-Name:value` — Headers and values are `:` separated
* `jsonKey=value` — Sends `{"jsonKey": "value"}` in the `POST` or `PUT` body. This will also automatically set the `Content-Type` and `Accept` headers to `application/json`.
* `jsonKey:=[1,2,3]` — Allows you to specify raw JSON data, sending `{"jsonKey": [1, 2, 3]}` in the body.

### Limitations

- You cannot send `multipart/mime` (file upload) data.
- Client certs are not supported.
- Server certs cannot be verified.
- Output cannot be customized.
- Responses are not syntax highlighted.

## Guzzle Middleware

This package provides three different middleware handlers:

* `\Akamai\Open\EdgeGrid\Handler\Authentication` - provides transparent API request signing
* `\Akamai\Open\EdgeGrid\Handler\Verbose` - easily output (or log) responses
* `\Akamai\Open\EdgeGrid\Handler\Debug` - easily output (or log) errors

All three can be added transparently when using the `Client`, to a standard `\GuzzleHttp\Client`, or as a handler.

### Transparent Use

To enable `Authentication` call `Client->setAuthentication()`, or pass in an instance of `\Akamai\EdgeGrid\Authentication`
to `Client->__construct()`.

To enable `Verbose` call `Client->setInstanceVerbose()` or `Client::setVerbose()` passing in on of `true|resource|[resource output, resource error]. Defaults to `[STDOUT, STDERR]`.

To enable `Debug` call `Client->setInstanceDebug()`, `Client::setDebug()`, or set the `debug` config option with `true|resource`. Defaults to `STDERR`.

### Middleware

#### Authentication Handler

```php
// Create the Authentication Handler
$auth = \Akamai\Open\EdgeGrid\Handler\Authentication::createFromEdgeRcFile();
// or:
$auth = new \Akamai\Open\EdgeGrid\Handler\Authentication;
$auth->setAuth($client_token, $client_secret, $access_token);

// Create the handler stack
$handlerStack = \GuzzleHttp\HandlerStack::create();

// Add the Auth handler to the stack
$handlerStack->push($auth);

// Add the handler to a regular \GuzzleHttp\Client
$guzzle = new \GuzzleHttp\Client([
    "handler" => $handlerStack
]);
```

#### Verbose Handler

```php
// Create the handler stack
$handlerStack = HandlerStack::create();

// Add the Auth handler to the stack
$handlerStack->push(new \Akamai\Open\EdgeGrid\Handler\Verbose());

// Add the handler to a regular \GuzzleHttp\Client
$guzzle = new \GuzzleHttp\Client([
    "handler" => $handlerStack
]);
```

### Debug Handler

```php
// Create the handler stack
$handlerStack = HandlerStack::create();

// Add the Auth handler to the stack
$handlerStack->push(new \Akamai\Open\EdgeGrid\Handler\Debug());

// Add the handler to a regular \GuzzleHttp\Client
$guzzle = new \GuzzleHttp\Client([
    "handler" => $handlerStack
]);
```

### Using PHP 5.3 (not recommended)

> PHP 5.3 has been EOL since August 14th, 2014, and has **known** security vulnerabilities, therefore we do not recommend using it.
> However, we understand that many actively supported LTS distributions are still shipping with PHP 5.3, and therefore we are providing
> the following information.

The signer itself is PHP 5.3 compatible and has been moved to the [akamai-open/edgegrid-auth](https://packagist.org/packages/akamai-open/edgegrid-auth) package.

## Author

Davey Shafik <dshafik@akamai.com>

## License

Copyright © 2022 Akamai Technologies, Inc. All rights reserved

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at <http://www.apache.org/licenses/LICENSE-2.0>.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[Akamai {OPEN} EdgeGrid Authentication]: https://developer.akamai.com/introduction/Client_Auth.html