---
url: >-
  https://viktorprogger.github.io/posts/dont-do-this-non-existent-trait-fields.md
description: A code example from a real project where trait uses its descendant fields
---
With this article, I open a rubric "Don't do this." It will contain code from real projects, from which I will remove all references to the projects themselves. And, of course, my explanations of why shouldn't you do this, and the way it
should be done.

So, the first patient. Inside the trait, the fields of its descendants are used. I mean something like this:

```php
trait Foo
{
    public function bar(): string
    {
        return $this->field . static::CONSTANT . $this->method();
    }
}
```

:::warning
Or even worse, `$this->{$attribute}`. But it's out of the question, as the chance of getting an error 500 rises to almost 100%.
:::

## What's wrong here?

Technically, a class that uses this trait is not required to declare these field, constant and method. *They may not be there*. At the same time, you can even deceive the static analyzer if you add the following comment to the trait:

```php
/**
  * @mixin SomeClass
  */
```

Of course, this will work only if `SomeClass` contains them all: the `$field`, the `CONSTANT` and the `method()`.

The problem arises when this trait is added to another class and the developer does not know or forgets to declare all this stuff.

:::note
As a rule, traits of such a quality contain lots of code: 500-1000 lines or even more. It becomes quite
difficult to find out such errors. And given that the project is unlikely to be covered by tests, the problem may well reach production environment and cause problems for the end users.
:::

## What is the right way?

One should use abstract methods:

```php
trait Foo
{
    public function bar(): string
    {
        return $this->getField() . $this->getConstant() . $this->method();
    }

    abstract protected function getField(): string;
    abstract protected function getConstant(): string;
    abstract protected function method(): string;
}
```

Thus, we have a technical limitation at the language level: any class that includes this trait will also be **obliged** to declare three methods on which the main functionality of the trait depends.

:::tip
Just do not forget to add detailed comments to the methods about what values it should return and what exceptions should be thrown in which case.
:::

## Correct option 2

*For adherents of the ideas "Traits are evil" and "Composition is strength"*:

```php
interface BarInterface
{
    public function getField(): string;
    public function getConstant(): string;
    public function method(): string;
}

readonly final class Foo
{
    public function bar(BarInterface $bar): string
    {
        return $bar->getField() . $bar->getConstant() . $bar->method();
    }
}
```

Here we replace the trait with two things:

* Interface with the three methods which were `abstract` and `protected` in the trait.
* The class to which we pass the implementation of this interface in order to call these methods and use their execution results.

:::tip
The result seems to be the same, but there is a nuance. Now the classes that implement `BarInterface` are not linked by common code.
This reduces the code coupling and increases its cohesion.
Further elaboration and refactoring of a such code will be more simple and comfortable.
:::
