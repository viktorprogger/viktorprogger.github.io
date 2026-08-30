---
url: 'https://viktorprogger.github.io/posts/elusive-502-gateway-timeout.md'
description: >-
  A live story about our product to get 502 error, and how did we walked around
  the bush
---
What are the worst mistakes in programming? I would single out two types: 

* those that cause a business to lose a lot of money
* and those that are the least common. 

While the first ones are immediately clear, what about the second ones? The fact is that the less often we encounter some type of error, the more difficult it is to understand what caused them.

This happened to me at work as well. One morning, the testers noticed that some requests to the backend returned a 502 (Gateway Timeout) error. This bug was a release stopper, so all the senior developers and a devops engineer took it up. At first, it was believed that Nginx returned this error, and the backend had nothing to do with it. After a while, we realized that PHP was to blame. We sinned for everything: turned off the BlackFire, changed OpCache settings, tried different patch versions in PHP, and so on. However, the error itself was not in the infrastructure, but directly in the application code.

### Foreword

As an introduction, I’ll tell you that the application is based on the yii2 framework and is originally written by not the most intelligent developers. For example, the DI container was not used, instead there were components, as it was originally intended in the framework many years ago. When the project took off, specialists of a higher level came to it, and they began to write more architecturally verified code. Because of one such change, we got an error that looked like a 502 on the front-end, and for which there were no records in any logs. The first thing that came to mind: this situation with the logs occurs due to memory overflow. That is, when the php process uses more memory than it is allocated by the system. We are familiar with this situation and know how to recognize it, that is why we checked it right away. But we didn't hit this time.

In some time we turned out, that our problem was of a similar nature: the process fell off due to reaching another system limitation, not memory. In a very non-obvious place, we reached the limit on the maximum nesting of function calls. In other words, we have entered a recursion. Moreover, this recursion was implicit. That is, the function did not call itself, but another function, which in turn called the first function, and that again the second, and that again the first one...

And we came to this as follows. In the configuration, we had a component (let's call it 'foo' for clarity). The configuration of this component looked like this:

```php
'foo' => [
    'class' => FooClass::class,
    'property1' => 'value1',
]
```

One of our developers added the definition of this class to the DI container:

```php
FooClass::class => static fn() => Yii::$app->get('foo'),
```

What did we get as a result? When the project code accessed a component, the framework tried to create that component. Seeing the need to get a class object in the component definition, it looked for this class first in the DI container, where it found the definition `Yii::$app->get('foo')`, and again tried to get the component. And so on in a circle.

This is how the simplest mistake managed to fool developers with more than 10 years of experience. We just don't see it very often.
