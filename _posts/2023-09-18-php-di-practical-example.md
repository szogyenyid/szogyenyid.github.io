---
layout: post
title: "Mastering Dependency Injection in PHP with PHP-DI: A practical example"
date: 2023-09-18 17:00:00 +0200
categories:
    - php
    - learning
tags:
    - programming
    - php
    - di
    - dependency-injection
    - php-di
    - design-patterns
description: "Discover the world of Dependency Injection (DI) in PHP through an example. From crafting a basic "Hello World" script to mastering advanced techniques, you'll see how DI enhances flexibility, maintainability, and code organization with the help of PHP-DI, a powerful dependency injection container. Whether you're a PHP beginner or an experienced developer, this guide equips you to create clean, modular, and efficient PHP applications."
#last_modified_at: 2023-09-08 10:38:00 +0200
author: Daniel Szogyenyi
readtime: 10
---

## Introduction

Dependency injection is a crucial concept in modern PHP development, facilitating flexibility, testability, and maintainability of your code. In this comprehensive guide, we will explore dependency injection through practical examples and dive deep into the powerful PHP-DI library. By the end, you'll have a solid understanding of how to use DI containers effectively in your PHP projects.

## Simple Hello World app

### Building the Foundation

We'll begin with the most basic setup – a simple "Hello World" application. First, create a basic PHP project using Composer:

```shell
composer init
```

Now, let's write a minimal PHP script that prints "Hello World!" to the console:

```php
<?php

include __DIR__ . '/../vendor/autoload.php';

echo "Hello World!";
```

In this script, we include the Composer-generated autoload file to load any dependencies. Then, we use `echo` to display our greeting message.

This straightforward script demonstrates the essence of PHP without any complexity. However, real-world applications often require more structure and organization.

## Evolving into Object-Oriented Code

In software development, object-oriented programming (OOP) provides a powerful way to structure and organize code. Let's take a step towards OOP by encapsulating our "Hello World" logic within a class:

```php
class App
{
    public function run()
    {
        echo 'Hello World!';
    }
}

$app = new App();
$app->run();
```

Here, we define a class called `App` with a `run` method, then create an actuak instance of App and call single its method.

While this may seem like an unnecessary abstraction for such a simple task, it sets the stage for more significant enhancements and customization down the road.

## Customization with strings

As applications grow, they often require customization and configuration options. To facilitate this, we can introduce constructor parameters. These parameters allows us to configure our classes from an external source when instantiating, making them more flexible and configurable.

Let's enhance our `App` class to accept custom parameters for our greeting:

```php
class App
{
    public function __construct(
        private string $function = 'echo',
        private string $greeting = 'Hello',
        private string $name = 'World'
    ) {
    }
    public function run()
    {
        echo 'Hello World!';
        if ($this->function === 'echo') {
            echo $this->greeting . ' ' . $this->name . '!' . PHP_EOL;
        } else {
            throw new InvalidArgumentException('Invalid function');
        }
    }
}

$app = new App(
    name: "szogyenyid"
);
$app->run();
```

Now, the `App` class accepts three optional string parameters: `$function`, `$greeting`, and `$name`. These parameters allow you to customize the behavior of the application. For example, you can change the greeting or target a different name to greet.

This customization is a significant improvement over the initial "Hello World" script, but as your application grows, managing dependencies can become challenging.

## Introducing Interfaces and Implementations

In large applications, it's essential to adhere to the principles of separation of concerns and dependency inversion. One way to achieve this is by using interfaces and multiple implementations. Let's introduce the concepts of `WriterInterface` and `GreeterInterface`.

```php
interface WriterInterface
{
    public function write(string $message): void;
}

interface GreeterInterface
{
    public function __construct(WriterInterface $writer);
    public function greet($name): void;
}
```

Here, we define two interfaces: `WriterInterface` and `GreeterInterface`. The `WriterInterface` specifies a method `write`, which any writer class must implement. The `GreeterInterface` specifies a constructor that accepts a `WriterInterface` dependency and a `greet` method.

Next, we create two implementations of these interfaces:

```php
class Echoer implements WriterInterface
{
    public function write(string $message): void
    {
        echo "Echo: " . $message . PHP_EOL;
    }
}

class Helloer implements GreeterInterface
{
    public function __construct(
        private WriterInterface $writer
    ) {
    }

    public function greet($name): void
    {
        $this->writer->write("Hello $name!");
    }
}
```

The `Echoer` class implements the `WriterInterface` by echoing the message with a `"Echo: "` prefix. The `Helloer` class implements the `GreeterInterface` and uses the injected `WriterInterface` to greet a person.

Finally, we update our `App` class to accept a `GreeterInterface` dependency:

```php
class App
{
    public function __construct(
        private GreeterInterface $greeter
    ) {
    }

    public function run(string $name)
    {
        $this->greeter->greet("$name");
    }
}

// Usage example:
$app = new App(
    greeter: (new Helloer((new Echoer()))),
);
$app->run("szogyenyid");
```

Now, our `App` class depends on a `GreeterInterface`, which can be any implementation of that interface. In this example, we create an instance of the `Helloer` class and inject it as the greeter for our `App`. This approach allows us to easily switch between different greeters or writers without modifying the `App` class itself.

However, it's clearly visible that a chain of multiple dependency injections can result in an unreadable mess (imagine both classes taking four objects instead of a single one).

## Leveraging PHP-DI for Dependency Injection

While manually managing dependencies works well for small projects, it becomes cumbersome as your application grows. This is where PHP-DI comes to the rescue. PHP-DI is a powerful dependency injection container that can automate the injection of dependencies based on type hinting.

To use PHP-DI, start by installing it through Composer:

```shell
composer require php-di/php-di
```

Next, create a PHP-DI container and specify the dependencies and their implementations:

```php
$container = new \DI\Container([
    GreeterInterface::class => \DI\autowire(Helloer::class),
    WriterInterface::class => \DI\autowire(Echoer::class),
]);

$app = $container->get(App::class);
$app->run("szogyenyid");
```

In this example, we configure the container to use `Helloer` as the implementation for `GreeterInterface` and `Echoer` as the implementation for `WriterInterface`. When we request an instance of the `App` class from the container, PHP-DI automatically resolves its dependencies based on type hinting.

PHP-DI simplifies dependency management, promotes cleaner code, and reduces the need for manual wiring of dependencies.

## Advanced Dependency Injection with PHP-DI

While PHP-DI's autowiring is convenient, there are situations where you may need to explicitly define dependencies. Let's explore this by adding another writer implementation and using it in the `Helloer` class.

### Adding another writer

First, create a new writer implementation called `Printer`:

```php
class Printer implements WriterInterface
{
    public function write(string $message): void
    {
        printf("Print: %s\n", $message);
    }
}
```

The `Printer` class prints messages using the `printf` function.

### Using the new writer

Now, let's modify the `Helloer` class to explicitly tell PHP-DI to use the `Printer` implementation:

```php
class Helloer implements GreeterInterface
{
    public function __construct(
        #[Inject(Printer::class)]
        private WriterInterface $writer
    ) {
    }
    // (...)
}
```

By adding the `#[Inject(Printer::class)]` attribute, we tell PHP-DI to use the `Printer` implementation for the `WriterInterface` dependency within the `Helloer` class. This explicit definition overrides the default autowiring.

This flexibility allows you to fine-tune dependency resolution and cater to specific use cases in your application.

## Conclusion

In this guide, we explored the power of dependency injection (DI) in PHP, from basic "Hello World" scripts to advanced, modular code. DI empowers developers to build flexible and maintainable applications.

We began with object-oriented principles, enhancing code structure. Dependency injection allowed easy customization, adapting the application's behavior effortlessly.

Interfaces and multiple implementations improved modularity and code maintainability. PHP-DI emerged as a valuable tool, streamlining dependency management.

Advanced DI techniques offered fine-grained control over dependency resolution.

Mastering DI and PHP-DI equips you to build efficient PHP applications. Embrace DI for adaptable and maintainable code in projects of all sizes.

Happy coding and may your PHP applications thrive with these newfound skills!
