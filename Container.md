# Illusion Container

- [Introcuction](#introduction)
- [Basic Concepts](#basic-concepts)
- [Basic Usage](#basic-usage)
- [Using "Array Access" and "Magic Methods"](#using-array-access-and-magic-methods)
- [Services](#services)
- [Instances](#instances)
- [Shared Instances](#shared-instances)
- [Protecting Parameters](#protecting-parameters)
- [Method Injection](#method-injection)
- [Extending Services](#extending-services)
- [Helper Functionality](#helper-functionality)

## Introduction

The Illusion Container is a Dependency Injection Container and it was highly inspired by [Laravel's IoC Container](http://laravel.com/docs/5.0/container) and [Pimple](https://github.com/silexphp/Pimple).

## Basic Usage

In order to use the Illusion Container you have to create an instance of the container and then register a binding, such as:

```php
use Illusion\Container\Container;

// Instantiate the container
$container = new Container();

// Register the service Foo, binding it to 'foo' in the container.
$container->register('foo', function($c) {
    return new Foo;
});

// Resolve the 'foo' binding.
$foo = $container->resolve('foo');
```
And now the `$foo` variable is an instance of the `Foo` service.

Let's look at the following example:

```php
class Foo
{
    public function give($amount)
    {
        return $amount;
    }
}

class Bar
{
    protected $value;

    public function __construct(Foo $foo)
    {
        $this->value = $foo->give(1);
    }

    public function get()
    {
        return $this->value;
    }
}

// Using the container to bind the Bar service
$container->register('bar', 'Bar');

// Resolving the binding.
$bar = $container->resolve($bar);

echo $bar->get(); // Outputs: 1
```

As you can see from the example above, the container automatically injects any dependency your constructor needs to get instantiated.

> **Note:** Everytime you resolve a binding, a new instance of the service will be given to you.
> If your objective is to get the same instance after each resolve, check [Shared Instances](#shared-instances).


## Using "Array Access" and "Magic Methods"

You can use the "Array Access" and "Magic Methods" to access the basic functionality of the container.

```php
// Registering:
    $container->register('foo', 'Foo'); // Normal Call
    $container->foo = 'Foo';            // Magic Method
    $container['foo'] = 'Foo';          // Array Access

// Resolving:
    $container->resolve('foo');         // Normal Call
    $container->foo;                    // Magic Method
    $containter['foo'];                 // Array Access

// If the binding is set:
    $container->has('foo');             // Normal Call
    isset($container->foo);             // Magic Method
    isset($container['foo']);           // Array Access

// Deleting a binding:
    $container->delete('foo');          // Normal Call
    unset($container->foo);             // Magic Method
    unset($container['foo']);           // Array Access
```
This eases the usage of your service methods such as:
```php
$container->foo->method();

// Instead of:
$container->resolve('foo')->method();
```

> **Note:** Registering a service through "Array Access" or "Magic Methods" will not create a [Shared Instance](#shared-instances).

# Services

As seen previously, in order to use a service with the container you do:
```php
// Register the service.
$container->register('foo', function($c) {
    return new \App\Foo;
});

// Resolve the service.
$container->resolve('foo');
```

> **Note:** The **$c** argument is the actual instance of the container.
> The last argument of the passed closures will always be an instance of the container.

You can also allow your service to depend on values passed through parameteres:

```php
$container->register('foo', function($message, $number, $c) {
    // Do operations with $number and $message
    return new \App\Foo;
});

$container->resolve('foo', array('Hello!', 20));
```

## Instances

If you already have an instance of your service, you can add it into the container such as:

```php
// Your instance
$foo = new Foo;

// Bind your already instantiated class into 'foo'.
$container->instance('foo', $foo);

// And then resolve it
$container->resolve('foo');
```

> **Important:** The instance() method creates a Shared Instance by default!

If don't want to create a [Shared Instance](#shared-instances) by default you can do:
```php
$container->instance('foo', $foo, false);
```

## Shared Instances

A Shared Instance in the container is an instance that only gets resolved once and then returns that same instance on every call.

So everytime you resolve the binding, you will always obtain the same instance.

To create a shared instance, all you have to do is the following:

```php
// Register a shared instance
$container->register('foo', 'Foo', true);

// Shortcut for register
$container->singleton('foo', 'Foo');

// Alias of singleton method
$container->share('foo', 'Foo');

// Or if you already have an instance:
$container->instance('foo', $fooInstance);
```

> **Note:** Both "Array Access" and "Magic Methods" do not allow you to create shared instances.

## Protecting parameters

The Illusion Container also allows you to protect parameters.

This can be useful in case you want to store configuration, or maybe even functions.

```php
// Register a protected parameter
$container->protect('factorial', function($number) {
    // Calculate the factorial of a number and return it..
});

// Resolve a protected parameter
$container->getProtected('factorial', array(5));
```

Since the container will always get injected into the last parameter of the closure, you can use it within your protected parameter:

```php
// Register a protected parameter
$container->protect('factorial', function($number, $container) {
    // Use the container to resolve or register services
    // Calculate the factorial of a number and return it..
}

// Resolve a protected parameter
$container->getProtected('factorial', array(5));
```

## Method Injection

Like Constructor Injection, the Illusion Container also lets you do Method Injection.

```php
class Bar {}

class Foo
{
    protected $bar;
    public function set(Bar $bar)
    {
        $this->bar = $bar;

        return 'Bar has been set!';
    }
}

$result = $container->method('Foo@set');

echo $result; // Outputs: Bar has been set!
```

## Extending Services

If you want to extend an already defined service, you can by using the `extend()` method.

```php
// Defining a service:
$container->register('foo', 'Foo');

// Resolving the service
$container->resolve('foo');

// Extending the 'foo' service
$container->extend('foo', function ($foo, $container) {
    // Do something with the container
    // Do something with $foo
    return $foo;
});

// Use the extended service:
$container->resolve('foo');
```

## Helper Functionality

### Method keys()
    This method returns an array with all of the registered keys within the container.

### Method deleteInstance($key)
    Deletes an Shared Instance of a key binding.
    
    Useful if you want to resolve your Shared Instance again.

### Method deleteInstances()
    Deletes all Shared Instances from the container.
    
    Useful if you want to resolve all of your Shared Instances again.

### Method flush()
    Deletes all of the keys within the container.
