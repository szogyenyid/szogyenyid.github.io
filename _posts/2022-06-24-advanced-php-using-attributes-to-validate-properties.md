---
layout: post
title: "Advanced PHP: Using attributes to validate property values"
date: 2022-06-24 18:47:00 +0200
categories: # Learning | Data Science | Security | Meta | Stories | PHP
    - php
    - learning
tags:
    - programming
    - php
    - attributes
    - php8
description: # TODO
last_modified_at: 2022-06-25 18:49:00 +0200
author: Daniel Szogyenyi
readtime: 4
---

# About attributes

As of PHP8, developers are able to use **attributes**, which are an extremely powerful addition to a language. The concept is not new, docbloc annotations were used to achive the same functionality. However, with the native support of attributes, now it's easier to add and use metadata to our code than anytime before. In this article, I will show you a practical example of the usage of **PHP Attributes**.<br><br>

## What are Attributes?

According to the [Manual][php-manual-attributes]:
> <span>Attributes offer the ability to add **structured, machine-readable metadata information** on declarations in code: Classes, methods, functions, parameters, properties and class constants can be the target of an attribute. The metadata defined by attributes can then be inspected at runtime using the **Reflection APIs**. Attributes could therefore be thought of as a configuration language embedded directly into code.</span>

# Implementing a practical example: property validation

## The Idea

There are several use cases, when it would be beneficial to validate the values of properties. One example is email strings - by default, that can be handled by PHP's `filter_var()` function -, especially the special ones, like "email addresses that end with @case-solvers.com". The same can be applied to numbers (eg. it must be between two constant values). 

## Expected output

These kind of validations can be achieved in an easy and maintainable way with the use of attributes. In the following example, I'd like to achieve something like this:
    
```php
class Colleague
{
    #[Format("/^[a-z0-9+.]+@case-solvers\.com$/")]
    private string $email;
    
    #[Between(16, 70)]
    private int $age;

    public function setEmail(string $email)
    {
        if (AttributeValidator::validate(self::class, "email", $email)) {
            $this->email = $email;
        } else {
            throw new Exception("Validation failed.");
        }
    }
    public function setAge(int $newAge)
    {
        if (AttributeValidator::validate(self::class, "age", $newAge)) {
            $this->age = $newAge;
        } else {
            throw new Exception("Validation failed.");
        }
    }
}
```
In this case, the property `$email` must look like a valid company e-mail address of Case Solvers, and `$age` has to be between 16 and 70.

## Defining validator interfaces

As attributes can hold any type of metadata, and we only want to handle validations,  it would be a good idea to define an interface for validator classes (`Format` and `Between` in this case). As the type of the parameter may change, I will use separate interfaces for each type.

```php
interface StringValidator
{
    public function validate(string $value): bool;
}
interface NumericValidator
{
    public function validate(float $value): bool;
}
```

We could define separate `IntValidator` and `FloatValidator` classes if they would be necessary. To keep it simple, I went with a single `NumericValidator` interface, and will let PHP cast integers to float.

## Defining the Validator classes

Now it's defined that each Validator class must have a `validate($value)` method, so let's implement the `Format` and `Between` classes.

As attributes can only be used in PHP8, we are able to use another new feature: contructor property promotion. Also, each attribute class must have an attribute, defining itself as... an attribute. As I want my validators to work only on class properties, I will define a target too.

```php
#[Attribute(Attribute::TARGET_PROPERTY)]
class Between implements NumericValidator
{
    public function __construct(
        private float $lowerBound,
        private float $upperBound
    ) { }
    
    public function validate(float $value): bool
    {
        return ($this->lowerBound < $value) && ($value < $this->upperBound);
    }
}
```

```php
#[Attribute(Attribute::TARGET_PROPERTY)]
class Format implements StringValidator
{
    public function __construct(
        private string $regex,
    ) { }
    
    public function validate(string $string): bool
    {
        return preg_match($this->regex, $string);
    }
}
```

As you can see, attribute classes are like any other class, developers can implement any complex algorithms they want. But keep in mind, there's a chance these classes will be instantiated (and `validate` will be called) multiple times, so it's better to keep them as simple as possible.

## Implemeting the class that actually validates the values

As attributes are only pieces of metadata added to the code, their content is not automatically instantiated and used at runtime - we need to do so manually. For this purpose, I will create a simple class named `AttributeValidator`.

```php
class AttributeValidator
{
    public static function validate(
        string $class,
        string $propertyName,
        mixed $value
    ): bool {

        // Get a reflection of the property, so we will be able to get the attributes.
        $ref = new ReflectionProperty($class, $propertyName);

        foreach ($ref->getAttributes() as $attribute) {
            $validator = $attribute->newInstance();
            // Only pass values to the corresponding typed validators.
            if ($validator instanceof StringValidator && is_string($value)) {
                if (!$attribute->newInstance()->validate($value)) {
                    return false;
                }
            } elseif ($validator instanceof NumericValidator && is_numeric($value)) {
                if (!$attribute->newInstance()->validate($value)) {
                    return false;
                }
            }
        }

        return true;
    }
}
```

As seen, if a property has multiple validator attributes, all of them will be evaluated, and if any of them fails, the whole validation will fail.

## Using the AttributeValidator

At this point, the `AttributeValidator` is ready to be used, and it is ready to handle any new `StringValidator` or `NumericValidator` attributes we write in the future. Keep in mind, if you define new interfaces, you need to handle them in `AttributeValidator::validate()` as well.

# Conclusion

Attributes are a powerful addition to the PHP language. They are a way to add metadata to your code, and they are a way to define validation rules for your properties.

[php-manual-attributes]: https://www.php.net/manual/en/language.attributes.overview.php
