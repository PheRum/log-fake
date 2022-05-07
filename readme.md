<p align="center"><img src="/art/header.png" alt="Log Fake: a Laravel package by Tim MacDonald"></p>

# Log fake for Laravel

![CI](https://github.com/timacdonald/log-fake/workflows/CI/badge.svg) [![codecov](https://codecov.io/gh/timacdonald/log-fake/branch/master/graph/badge.svg)](https://codecov.io/gh/timacdonald/log-fake) [![Mutation testing badge](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fbadge-api.stryker-mutator.io%2Fgithub.com%2Ftimacdonald%2Flog-fake%2Fmaster)](https://dashboard.stryker-mutator.io/reports/github.com/timacdonald/log-fake/master) [![Total Downloads](https://poser.pugx.org/timacdonald/log-fake/downloads)](https://packagist.org/packages/timacdonald/log-fake)

A bunch of Laravel facades / services are able to be faked, such as the Dispatcher with `Bus::fake()`, to help with testing and assertions. This package gives you the ability to fake the logger in your app, and includes the ability to make assertions against channels, stacks, and a whole bunch more introduced in logging overhaul in Laravel `5.6`.

## Version support

- **PHP**: 8.0, 8.1
- **Laravel**: 9.0
- **PHPUnit**: 9.0

You can find support for older versions in [previous releases](https://github.com/timacdonald/log-fake/releases).

## Installation

You can install using [composer](https://getcomposer.org/) from [Packagist](https://packagist.org/packages/timacdonald/log-fake).

```sh
composer require timacdonald/log-fake --dev
```

## Basic usage

```php
public function testItLogsWhenAUserAuthenticates()
{
    /*
     * Test setup.
     *
     * In the setup of your tests, you can call the following `bind` helper,
     * which will switch out the underlying log driver with the fake.
     */
    LogFake::bind();

    /*
     * Application implementation.
     *
     * In your application's implementation, you then utilise the logger, as you
     * normally would.
     */
    Log::info('User logged in.', ['user_id' => $user->id]);

    /*
     * Test assertions.
     *
     * Finally you can make assertions against the log channels, stacks, etc. to
     * ensure the expected logging occurred in your implementation.
     */
    Log::assertLogged('info', function ($message, $context) {
        return $message === 'User logged in.'
            && $context === ['user_id' => 5];
    });
}
```

## Channels

If you are logging to a specific channel (i.e. not the default channel) in your app, you need to also prefix your assertions in the same manner.

```php
public function testItLogsWhenAUserAuthenticates()
{
    // setup...
    LogFake::bind();

    // implementation...
    Log::channel('slack')->info('User logged in.', ['user_id' => $user->id]);

    // assertions...
    Log::channel('slack')->assertLogged('info', function ($message, $context) {
        return $message === 'User logged in.' 
            && $context === ['user_id' => 5];
    });
}
```

## Stacks

If you are logging to a stack in your app, like with channels, you will need to prefix your assertions. Note that the order of the stack does not matter.

```php
public function testItLogsWhenAUserAuthenticates()
{
    // setup...
    LogFake::bind();

    // implementation...
    Log::stack(['stderr', 'single'])->info('User logged in.', ['user_id' => $user->id]);

    // assertions...
    Log::stack(['stderr', 'single'])->assertLogged('info', function ($message, $context) {
        return $message === 'User logged in.' 
            && $context === ['user_id' => 5];
    });
}
```

That's it really. Now let's dig into the available assertions to improve you experience testing your applications logging.

## Available assertions

Remember that all assertions are relative to the channel or stack as shown above.

- [`assertLogged()`](#assertlogged)
- [`assertLoggedTimes()`](#assertloggedtimes)
- [`assertNotLogged()`](#assertnotlogged)
- [`assertNothingLogged()`](#assertnothinglogged)
- [`assertWasForgotten()`](#assertwasforgotten)
- [`assertWasForgottenTimes()`](#assertwasforgottentimes)
- [`assertWasNotForgotten()`](#assertwasnotforgotten)
- [`assertChannelIsCurrentlyForgotten()`](#assertchanneliscurrentlyforgotten)
- [`assertCurrentContext()`](#assertcurrentcontext)

### assertLogged()

Assert that a log was created.

#### Can be called on...

- [x] Facade base (default channel)
- [x] Channels
- [x] Stacks

#### Example tests...

```php
/*
 * implementation...
 */

Log::info('User logged in.');

/*
 * assertions...
 */

Log::assertLogged(fn ($level, $message, $context) =>
    $message === 'User logged in.'
); // ✅

Log::assertLogged(fn ($level, $message, $context) =>
    $level === 'critical'
); // ❌ as log had a level of `info`.
```

### assertLoggedTimes()

Assert that a log was created a specific number of times.

#### Can be called on...

- [x] Facade base (default channel)
- [x] Channels
- [x] Stacks

#### Example tests...

```php
/*
 * implementation...
 */

Log::info('Stripe request initiated.');

Log::info('Stripe request initiated.');

/*
 * assertions...
 */

Log::assertLoggedTimes(fn ($level, $message, $context) =>
    $message === 'Stripe request initiated.',
    2
); // ✅

Log::assertLoggedTimes(fn ($level, $message, $context) =>
    $message === 'Stripe request initiated.',
    99
); // ❌ as the log was created twice, not 99 times.
```

### assertNotLogged()

Assert that a log was never created.

#### Can be called on...

- [x] Facade base (default channel)
- [x] Channels
- [x] Stacks

#### Example tests

```php
/*
 * implementation...
 */

Log::info('User logged in.');

/*
 * assertions...
 */

Log::assertNotLogged(fn ($level, $message, $context) =>
    $level === 'critical'
); // ✅

Log::assertNotLogged(fn ($level, $message, $context) =>
    $level === 'info'
); // ❌ as the level was `info`.
```

### assertNothingLogged()

Assert that no logs were created.

#### Can be called on...

- [x] Facade base (default channel)
- [x] Channels
- [ ] Stacks

#### Example tests

```php
/*
 * implementation...
 */

Log::channel('single')->info('User logged in.');

/*
 * assertions...
 */

Log::channel('stderr')->assertNothingLogged(); // ✅

Log::channel('single')->assertNothingLogged(); // ❌ as a log was created in the `single` channel.
```

### assertWasForgotten()

Assert that the channel / stack was forgotten at least one time.

#### Can be called on...

- [x] Facade base (default channel)
- [x] Channels
- [ ] Stacks

#### Example tests

```php
/*
 * implementation...
 */

Log::channel('single')->info('User logged in.');

Log::forgetChannel('single');

/*
 * assertions...
 */

Log::channel('single')->assertWasForgotten(); // ✅

Log::channel('stderr')->assertWasForgotten(); // ❌ as it was the `single` not the `stderr` channel that was not forgotten.
```

### assertWasForgottenTimes()

Assert that the channel / stack was forgotten a specific number of times.

#### Can be called on...

- [x] Facade base (default channel)
- [x] Channels
- [] Stacks

#### Example tests

```php
/*
 * implementation...
 */

Log::channel('single')->info('User logged in.');

Log::forgetChannel('single');

Log::channel('single')->info('User logged in.');

Log::forgetChannel('single');

/*
 * assertions...
 */

Log::channel('single')->assertWasForgottenTimes(2); // ✅

Log::channel('single')->assertWasForgottenTimes(99); // ❌ as the channel was forgotten twice, not 99 times.
```

### assertWasNotForgotten()

Assert that the channel / stack was not forgotten.

#### Can be called on...

- [x] Facade base (default channel)
- [x] Channels
- [ ] Stacks

#### Example tests

```php
/*
 * implementation...
 */

Log::channel('single')->info('User logged in.');

/*
 * assertions...
 */

Log::channel('single')->assertWasNotForgotten(); // ✅
```

### assertChannelIsCurrentlyForgotten()

Assert that a channel is _currently_ forgotten. This is distinct from [asserting that a channel _was_ forgotten](https://github.com/timacdonald/log-fake#assertwasforgotten).

#### Can be called on...

- [x] Facade base ~(default channel)~
- [ ] Channels
- [ ] Stacks

#### Example tests

```php
/*
 * implementation...
 */

Log::channel('single')->info('xxxx');

Log::forgetChannel('single');

/*
 * assertions...
 */

Log::assertChannelIsCurrentlyForgotten('single'); // ✅

Log::assertChannelIsCurrentlyForgotten('stderr'); // ❌ as the `single` channel was forgotten, not the `stderr` channel.
```

### assertCurrentContext()

Assert that the channel currently has the specified context. It is possible to provide the expected context as an array or alternatively you can provide a [truth-test closure](#truth-test-closures) to check the current context.

#### Can be called on...

- [x] Facade base (default channel)
- [x] Channels
- [ ] Stacks

#### Example tests

```php
/*
 * implementation...
 */

Log::withContext([
    'app' => 'Acme CRM',
]);

Log::withContext([
    'env' => 'production',
]);

/*
 * assertions...
 */

Log::assertCurrentContext([
    'app' => 'Acme CRM',
    'env' => 'production',
]); // ✅

Log::assertCurrentContext(fn ($context) => $context['app'] === 'Acme CRM')); // ✅

Log::assertCurrentContext([
    'env' => 'production',
]); // ❌ missing the "app" key.

Log::assertCurrentContext(fn ($context) => $context['env'] === 'develop')); // ❌ the 'env' key is set to "production"
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

## Advanced

### Custom messages in assertions

## Credits

- [Tim MacDonald](https://github.com/timacdonald)
- [All Contributors](../../contributors)

And a special (vegi) thanks to [Caneco](https://twitter.com/caneco) for the logo ✨

## Thanksware

You are free to use this package, but I ask that you reach out to someone (not me) who has previously, or is currently, maintaining or contributing to an open source library you are using in your project and thank them for their work. Consider your entire tech stack: packages, frameworks, languages, databases, operating systems, frontend, backend, etc.

## Upgrading

- Failure messages have been updated.
- assertLoggedTimes no longer has a default $times value.
- assertLogged no longer accepts an integer as the second parameter. Use `assertLoggedTimes` directly instead
- the `hasLogged` function has been removed. use assertions or the Log::logged(...)->isEmpty()" instead
- the `hasNotLogged` function has been removed. use assertions or the Log::logged(...)->isNotEmpty()" instead
- the `getLogger` method now returns a `ChannelFake`
- raw log arrays now contain the 'times_channel_has_been_forgotten_at_time_of_writing_log' key, indicating how many times the channel has been forgotten at the time of creation
`
- assertion closures now recieve an addition 3rd parameter int: times_forgotten
- Don't support named parameters
- The "stack:" prefix has been removed and now uses the channel name or the default value. channels are now comma seperated

- a stack never has "currentContext" as it is reset each time it is resolved from the manager.
- `getChannels` returns all channels - forgotten or not
- Note that exception messages could change at any time and are not protected and are not considered breaking changes.
