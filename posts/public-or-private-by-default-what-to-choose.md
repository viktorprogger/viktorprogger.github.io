---
url: >-
  https://viktorprogger.github.io/posts/public-or-private-by-default-what-to-choose.md
description: >-
  Explaining consequences of choosing one of these two approaches to be your
  default one
---
When I worked on another project, I met an interesting opinion. They say, all the code in the project should be as open as possible by default. I.e. we should make all new methods and fields `public`, and we should not put `final` anywhere, except some special cases when it's really necessary. It was motivated with the difficulties of further codebase support:

> If I want to inherit the class, and there is a private method, I have to study the code to understand why it was made `private` and whether I can make it `public`.

This opinion was very unusual for me, and I decided to figure it out if there is really more problems when fields and methods are private by default.\
Let's write an extra public method and see how easy is it to support it.

```php
interface FooInterface {
  public function foo();
}

class Foo implements FooInterface {
  public function foo() {
    // do stuff
    $this->bar();
    // do stuff
  }

  public function bar() {
    // do stuff
  }
}
```

Let us agree that at the time of writing the code, the `bar` method is no longer used anywhere, and technically we do not care whether to make it public or private.

## Private

If we make it private, we will encounter the voiced difficulties: before making the logic of this method public, the programmer will have to familiarize himself with the method code and find out if there are places specific to this particular class. For example, working with a state.

## Public

If you make the method public by default, we will avoid the likelihood that we will need to perform the work described above in the future. Just kidding, of course, there is no escape 😄

:::note
Nobody guarantees that a programmer will write the method strictly in such a way that it can be reused from any other place in the application. Especially since initially the method is not planned to be reused.
:::

Let's think about what other problems we can face in a few years after writing a such code. Let's imagine that we did not change this method of the year 3. And now we need to remove it or change the signature. The first question that arises: is this method used in other places of the project? Anyone of company developers could call a public method from any other place, and they all need to be found. You can say that modern IDEs solve this issue instantly, but what if the black magic of frameworks was used?

:::note
All these `__get()` and` __call()`, calling the method through an arbitrary string identifier, Yii2 behaviors, Laravel facades, etc... The presence of such approaches in the project makes the task a non-trivial one.
:::

Complicating it a little more: we found several places where the `bar` method was used, but it is not entirely clear why, and the authors of that code no longer work in your company. What to do in this case?

Let's add another potential problem to the piggy bank. Some developer decided to slightly change the functionality of `bar`, but did not deeply seek for usages of this method. And they were. But they were hidden.

## Conclusions

The public way does not solve the problem voiced for the private one, but adds a heap of new difficulties. Therefore, I definitely recommend that all fields, constants and methods of classes be made private if they are not used outside the class in which they are declared, and are not explicitly intended for this.

## An example of private-to-public refactoring

So, in our project there is a version of the `Foo` class with the private` bar` method. At some point, it became obvious that the functionality of this method was needed in a couple of another places. In most cases you can take `bar` to a separate class, which can be used anywhere in the project using your favorite DI Container. The full code will look like this:

```php
class Bar {
  public function bar() {
    // do stuff
  }
}

interface FooInterface {
  public function foo();
}

class Foo implements FooInterface {
  __construct(private Bar $bar) {}

  public function foo() {
    // do stuff
    $this->bar->bar();
    // do stuff
  }
}
```

In this case a public contract for the `bar` functionality will be concluded exactly at the moment when it became needed. Accordingly, the attitude towards it will be appropriate: as a consciously realized contract, and not "another public method, which is unlikely to be used anywhere." Because in such a project, nothing public is just done.
