<p align="center"><img src="/art/header.png" alt="Log Fake: a Laravel package by Tim MacDonald"></p>

# Log fake for Laravel

![CI](https://github.com/timacdonald/log-fake/workflows/CI/badge.svg) [![codecov](https://codecov.io/gh/timacdonald/log-fake/branch/master/graph/badge.svg)](https://codecov.io/gh/timacdonald/log-fake) [![Mutation testing badge](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fbadge-api.stryker-mutator.io%2Fgithub.com%2Ftimacdonald%2Flog-fake%2Fmaster)](https://dashboard.stryker-mutator.io/reports/github.com/timacdonald/log-fake/master) [![Total Downloads](https://poser.pugx.org/timacdonald/log-fake/downloads)](https://packagist.org/packages/timacdonald/log-fake)

A bunch of Laravel facades / services are able to be faked, such as the Dispatcher with `Bus::fake()`, to help with testing and assertions. This package gives you the ability to fake the logger in your app, and includes the ability to make assertions against channels and stacks introduced in logging overhaul in Laravel `5.6`.

## Version support

- **PHP**: 7.1, 7.2, 7.3, 7.4, 8.0
- **Laravel**: 5.6, 5.7, 5.8, 6.0, 7.0, 8.0
- **PHPUnit**: 7.0, 8.0, 9.0

## Installation

You can install using [composer](https://getcomposer.org/) from [Packagist](https://packagist.org/packages/timacdonald/log-fake).

```
$ composer require timacdonald/log-fake --dev
```

## Basic usage

```php
<?php

use Illuminate\Support\Str;
use TiMacDonald\Log\LogFake;
use Illuminate\Support\Facades\Log;

//...

LogFake::bind();

Log::info('Donuts have arrived');

Log::assertLogged('info', function ($message, $context) {
    return Str::contains($message, 'Donuts');
});
```

## Channels

If you are logging to a specific channel in your app, such as Slack with `Log::channel('slack')->critical('It is 5pm, go home')`, you need to also prefix your assertions in the same manner.

```php
<?php

use TiMacDonald\Log\LogFake;
use Illuminate\Support\Facades\Log;

//...

LogFake::bind();

Log::channel('slack')->alert('It is 5pm, go home');

Log::channel('slack')->assertLogged('alert'); // ✅ passes

// without the channel prefix...

Log::assertLogged('alert');  // ❌ fails
```

## Stacks

If you are logging to a stack in your app, like with channels, you will need to prefix your assertions. Note that the order of the stack does not matter.

```php
<?php

use TiMacDonald\Log\LogFake;
use Illuminate\Support\Facades\Log;

//...

LogFake::bind();

Log::stack(['bugsnag', 'sentry'])->critical('Perform evasive maneuvers');


Log::stack(['bugsnag', 'sentry'])->assertLogged('critical');  // ✅ passes

// without the stack prefix...

Log::assertLogged('critical'); // ❌ fails
```

## Available assertions

All assertions are relative to the channel or stack as shown in the previous examples.

### assertLogged($level, $callback = null)

```php
<?php

use Illuminate\Support\Str;

Log::assertLogged('info');

Log::channel('slack')->assertLogged('alert');

Log::stack(['bugsnag', 'sentry'])->assertLogged('critical');

// with a callback

Log::assertLogged('info', function ($message, $context) {
    return Str::contains($message, 'Donuts');
});

Log::channel('slack')->assertLogged('alert', function ($message, $context) {
    return Str::contains($message, '5pm');
});

Log::stack(['bugsnag', 'sentry'])->assertLogged('critical', function ($message, $context) {
    return Str::contains($message, 'evasive maneuvers');
});
```

### assertLoggedMessage($level, $message)

```php
<?php

Log::assertLoggedMessage('info', 'User registered');

Log::channel('slack')->assertLoggedMessage('alert', 'It is 5pm, go home');

Log::stack(['bugsnag', 'sentry'])->assertLoggedMessage('critical', 'Perform evasive maneuvers');
```

### assertLoggedTimes($level, $times = 1, $callback = null)

```php
<?php

use Illuminate\Support\Str;

Log::assertLoggedTimes('info', 5);

Log::channel('slack')->assertLoggedTimes('alert', 5);

Log::stack(['bugsnag', 'sentry'])->assertLoggedTimes('critical', 5);

// with a callback

Log::assertLogged('info', 5, function ($message, $context) {
    return Str::contains($message, 'Donuts');
});

Log::channel('slack')->assertLogged('alert', 5, function ($message, $context) {
    return Str::contains($message, '5pm');
});

Log::stack(['bugsnag', 'sentry'])->assertLogged('critical', 5, function ($message, $context) {
    return Str::contains($message, 'evasive maneuvers');
});
```

### assertNotLogged($level, $callback = null)

```php
<?php

use Illuminate\Support\Str;

Log::assertNotLogged('info');

Log::channel('slack')->assertNotLogged('alert');

Log::stack(['bugsnag', 'sentry'])->assertNotLogged('critical');

// with a callback

Log::assertNotLogged('info', function ($message, $context) {
    return Str::contains($message, 'Donuts');
});

Log::channel('slack')->assertNotLogged('alert' , function ($message, $context) {
    return Str::contains($message, '5pm');
});

Log::stack(['bugsnag', 'sentry'])->assertNotLogged('critical', function ($message, $context) {
    return Str::contains($message, 'evasive maneuvers');
});
```

### assertNothingLogged()

```php
<?php

Log::assertNothingLogged();

Log::channel('slack')->assertNothingLogged();

Log::stack(['bugsnag', 'sentry'])->assertNothingLogged();
```

## Inspection

Sometimes when debugging tests it's useful to be able to take a peek at the messages that have been logged. There are a couple of helpers to assist with this.

### dump($level = null)

```php
<?php

use TiMacDonald\Log\LogFake;
use Illuminate\Support\Facades\Log;

// ...

LogFake::bind();

Log::info('Donuts have arrived');

Log::channel('slack')->alert('It is 5pm, go home');

Log::dump();

// array:1 [
//   0 => array:4 [
//     "level" => "info"
//     "message" => "Donuts have arrived."
//     "context" => []
//     "channel" => "stack"
//   ]
// ]

Log::channel('slack')->dump();

// array:1 [
//   0 => array:4 [
//     "level" => "alert"
//     "message" => "It is 5pm, go home"
//     "context" => []
//     "channel" => "slack"
//   ]
// ]
```

### dd($level = null)

Works the same as `dump`, but also ends the execution of the test.

### dumpAll($level = null)

Only available calling on the default channel. This will dump every log regardless of the channel it was captured in.

```php
<?php

use TiMacDonald\Log\LogFake;
use Illuminate\Support\Facades\Log;

// ...

LogFake::bind();

Log::info('Donuts have arrived');

Log::channel('slack')->alert('It is 5pm, go home');

Log::dumpAll();

// array:1 [
//   0 => array:4 [
//     "level" => "info"
//     "message" => "Donuts have arrived."
//     "context" => []
//     "channel" => "stack"
//   ]
//   1 => array:4 [
//     "level" => "alert"
//     "message" => "It is 5pm, go home"
//     "context" => []
//     "channel" => "slack"
//   ]
// ]
```

## Credits

- [Tim MacDonald](https://github.com/timacdonald)
- [All Contributors](../../contributors)

And a special (vegi) thanks to [Caneco](https://twitter.com/caneco) for the logo ✨

## Thanksware

You are free to use this package, but I ask that you reach out to someone (not me) who has previously, or is currently, maintaining or contributing to an open source library you are using in your project and thank them for their work. Consider your entire tech stack: packages, frameworks, languages, databases, operating systems, frontend, backend, etc.
