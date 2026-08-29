---
url: 'https://viktorprogger.github.io/blog/posts/two-class-types-for-your-project.md'
---
The question of the area of responsibility of a particular class in a program architecture is no less important than architecture strategic planning.  Today I will tell you how to make this job easier for yourself.  We will solve a fairly large number of issues, leaving only two types of classes out of an infinite variety: DTO and Service. These types cover 90-99% of everything needed in any project. So, let's look at what they are, and why it's so good to leave only them.

::: info
Here and below, we will use an online store as an example.
:::

There are two main approaches to entities: the so-called "rich" and "anemic" models.  The rich model implies that the entity itself knows what can be done with it and how. Let's say we have such an entity as "order". In a rich domain model, its interface would look like this:

```php
interface OrderInterface
{
    public function getId(): OrderId;
    public function getProducts(): array;
    public function getCustomer(): Customer;
    public function getStatus(): OrderStatus;
    public function create(): void;
    public function update(): void;
    public function delete(): void;
    public function moveTo(OrderStatus $status): void;
    public function removeProduct(Product $product): void
    // etc.
}
```

It has getters and some methods for its changing: saving to a DB, status modifying (FSM?), etc.

The logic says there will be problems with the Single Responsibility and Interface Segregation principles (from the SOLID), and code coupling of such a project will be very high. Just imagine how many actions in other subsystems need to be done to create an Order: payments, logistics, warehouse, etc. Besides, I've never seen a successful appliance of a rich model. Maybe, I just don't know how to deal with it. If you have an example of a project in which it is justifiably applied, I'll be glad to see its code.

So, let's dwell on the anemic model in more detail. It is called so because the entity has no additional responsibilities and no knowledge of anything other than its own state. That is, if we take the example above, the Order will have only getters. We can even drop them since php 8.1:

```php
class Order
{
    public function __construct(
        public readonly OrderId $id;
        /** @var Product[] */
        public readonly array $products;
        public readonly Customer $customer;
        public readonly OrderStatus $status;
    ) {
    }
}
```

An entity in an anemic domain model is called DTO, Data Transfer Object.

::: note
There is the third approach, and it is really used in 99 cases of 100. It's a combination of the two above. There is no clear separation in class responsibility: entities contain business-logic, services has state. This is the most unpleasant option, because in support it becomes more and more expensive every day. It becomes more and more difficult to develop the system, programmers no longer have the pleasure of working with it, and nothing can be done about this situation without global refactoring.
:::

## Data Transfer Object

DTO is just a representation of some data, which travels from one place in a system to another. Ideally, a DTO is created at the entrance to the system and served at the exit from it. For example, a web controller: a user sends data, it comes to a controller in the form of a `$_POST` array, where we immediately form the corresponding DTO of it. To the internal services of the system we give a fulfilled DTO, not a raw data. Inside the system the data goes exclusively in the form of DTO.

There are two key features of DTO:

### Nothing except data representation

DTOs, by definition, lack any logic whatsoever. It is purely a representation of the data.
This approach allows DTO usage for end-to-end data transfer between different subsystems, including when this transfer is carried out over the network via API, queues, etc. Because it's just data, it's easy to be serialized and deserialized. The absence of business logic in data guarantees the absence of side effects from the transfer of an object to the next service or subsystem. It's very easy to support and change such classes.

### Immutability

DTOs are inherently immutable, and this is their main advantage. What for? Just imagine that a DTO can change.
Let's say we have code like this: `$service->foo($dto);` and we know that the `foo` method does not need to change the state of the DTO passed to it at all. But time passes, the code changes, and somewhere down the chain of calls, not in `foo` itself, something like this appeared: `$dto->bar = 'baz';`. How will the code that comes after the call to `foo` react to this? When will you notice the behavior has changed? How many hours will it take to debug?
Either we can immediately make the object immutable by any of these ways:

* mark all fields `readonly`
* make fields `private` and provide access via getters
* mark class `@psalm-immutable` if Psalm is used in your CI

*But what is the way to make changes to the data?*
No way. This is the point. If the business logic involves data changing, you must create a new object with this new data. Sometimes a DTO may imply such operations itself. To do so, [there are `with`-methods](https://github.com/yiisoft/docs/blob/master/010-code-style.md#immutable-methods) in the conventions of the YiiSoft team, that return a new object of the same class with changed data. Personally I'd prefer calling a variable `$instance` rather than `$new`, but that's a minor point.

## Service

The second type of classes needed in any project. In contrast to DTOs, services are stateless and implement some kind of logic. The example of services are repositories, controllers, factories, and other kinds of classes that *do something*. Often, services do some work on the data, that is, on the DTOs passed to them.\
What mistakes should be avoided when writing a service, and what benefits will we get?

### Service state

This is the only truly blunder in the design of any service. If a service store some state, there are high chances of getting unexpected side effects on the second call of the service. But if a service doesn't store any state, it can easily be reused elsewhere. Or in the same place, within the same php-process, but with other data. And this not only facilitates further development, but also makes it possible to use tools like RoadRunner and Swoole.\
So, the wrong way: `$service->setData($data)->foo()`, the right way: `$service->foo($data)`. If you still find it necessary to leave state in your service for some individual methods to work, it is likely that you should split this service into two.

It is worth making a remark about what the state of the class is. Its fields can contain various payloads: dependencies, settings, cache, and state.

* Dependencies are other services that allow the current one to do its job. We just follow the Dependency Inversion principle and deliver the dependencies to class through the constructor.
* Settings are values that allow the service to work. Database connection data for the repository, web address for the API client, cache prefix, etc.
* The runtime cache stores already calculated values in case these values do not change over time, and their recalculation is resource-intensive. Its usage looks like this:
  ```php
  if (!isset($this->cache[$id]) {
    $this->cache[$id] = $this->build($id);
  }

  return $this->cache[$id];
  ```
* The state is not set globally for the entire project/module, but changes depending on the context. This is the only kind of data in the service that can change during program execution. It is on it or with its help that the service performs some work.

So good services can have dependencies, settings and runtime cache, while DTOs only have state.

Of course, there are other architectural patterns that don't fit into these two categories. But they are needed much less frequently. For example, I needed to implement the Builder only once in 10 years of work.

***

And what non-DTO and non-service design patterns did you ever need to create? I'd like you to tell and discuss in [my telegram chat](https://t.me/viktorprogger_channel_ru), welcome!
