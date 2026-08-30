---
url: 'https://viktorprogger.github.io/posts/stateless-services-in-php.md'
description: Why should you use them and how to implement them in PHP
---
Imagine you've developed an order processing service that stores information about the current user and collects items in a shopping cart. Everything works perfectly in a simple web application where each request is handled by a separate PHP process. But one day, the project requirements change.

### The Trap of Long-Running Processes

Your application grows, and you decide to optimize its performance by switching to a modern web server like **RoadRunner**, **Swoole**, or **FrankenPHP**. Now, PHP processes don't die after each request but are *reused*. And here's where the interesting part begins: data from one user randomly "leaks" into the requests of another user because the service's state persists between requests.

Or imagine you decide to process orders asynchronously via a message queue. The worker processing the queue lives for a long time and handles thousands of messages in one lifecycle iteration. If your service maintains state, each unprocessed message can affect the processing of subsequent ones, creating tangled and hard-to-debug errors.

:::info
In such long-running processes, every line of code that saves state in class properties becomes a potential problem:

* State accumulates unnoticed, consuming more and more memory
* Parallel operations start conflicting with each other
* Debugging becomes a real quest because the service's behavior depends on its previous calls

:::

### Evolution of Usage

Even if your service is currently used in a simple context, without these trendy long-running states, its application might evolve:

* Code that was called once starts being called in a loop
* A simple HTTP request handler is moved to a console command
* Synchronous operations become asynchronous
* The need for scaling and parallel execution arises

In each of these cases, the presence of internal state becomes a source of problems and requires code refactoring.

## Solution: Stateless Services

A **Stateless service** is a class that does not store internal state between calls to its methods. Each method works only with the data passed to it as parameters.\
This approach allows abstracting away from the context in which the service is used.
A stateless service can be called once, in a loop, or in a long-running application.
It doesn't matter to it.

What are the alternatives? Services with state, or *stateful*. Let's consider an example I often see in various projects.

### Example of a Stateful Service

```php
class OrderProcessor
{
    private PaymentGateway $paymentGateway;
    private float $taxRate;
    private ?User $currentUser = null;
    
    public function __construct(PaymentGateway $paymentGateway, float $taxRate)
    {
        $this->paymentGateway = $paymentGateway;
        $this->taxRate = $taxRate;
    }

    public function setUser(User $user): void
    {
        $this->currentUser = $user;
    }

    public function processOrder(array $items): void
    {
        $total = $this->calculateTotal($items);
        $this->paymentGateway->charge($this->currentUser, $total);
    }

    private function calculateTotal(array $items): float
    {
        $subtotal = array_sum(array_map(
            fn($item) => $item['price'] * $item['quantity'],
            $items
        ));
        
        return $subtotal * (1 + $this->taxRate);
    }
}
```

In this example, it's essential to set the correct user as the service's state. And the result of the `processOrder()` method depends entirely on it. If, for some reason, the `setUser()` method was not called, it will attempt to charge money from the wrong person. What else could go wrong? Anything. The main reason is the human factor: you can be sure that sooner or later, one of the project's programmers will fail to ensure the correct user is set in the service, leading to a serious error. Here's what could happen technically:

* The method was forgotten to be called
* After setting the correct user, another user was set (e.g., during the processing of an event chain between the `setUser()` and `processOrder()` calls)
* After setting the correct user, the service instance was replaced

:::info
When a project is maintained for years, the probability of such an error increases to almost 100%. Meanwhile, fixing it would be very easy.
:::

### Example of a Stateless Implementation

To get rid of the state in the example above, simply remove the user field and the corresponding method, and add the user itself as a parameter to the method that actually depends on it:

```php
class OrderProcessor
{
    private PaymentGateway $paymentGateway;
    private float $taxRate;
    
    public function __construct(PaymentGateway $paymentGateway, float $taxRate)
    {
        $this->paymentGateway = $paymentGateway;
        $this->taxRate = $taxRate;
    }
    
    public function processOrder(User $user, array $items): void
    {
        $total = $this->calculateTotal($items);
        $this->paymentGateway->charge($user, $total);
    }

    private function calculateTotal(array $items): float
    {
        $subtotal = array_sum(array_map(
            fn($item) => $item['price'] * $item['quantity'],
            $items
        ));
        
        return $subtotal * (1 + $this->taxRate);
    }
}
```

### Advantages of This Approach

1. Predictability:
   * Method behavior depends only on input parameters
   * The same input always yields the same result (in this example, we always call `paymentGateway->charge()` with the same arguments as a result)
   * No implicit dependencies between calls

2. Ease of Testing:
   * No need to worry about initial state
   * Tests become simpler and clearer
   * Methods can be tested in isolation

3. Thread Safety:
   * Can be safely used in a multi-threaded environment
   * No problems with scaling
   * Absence of race conditions

## Understanding Different Data Types in Services

Not everything stored in class properties is harmful state. Let's break down three fundamentally different types of data:

### 1. Configuration

These are immutable values that define the service's behavior:

```php
class OrderProcessor
{
    private readonly float $taxRate;
    private readonly float $minimumOrderAmount;
    
    public function __construct(float $taxRate, float $minimumOrderAmount)
    {
        $this->taxRate = $taxRate;
        $this->minimumOrderAmount = $minimumOrderAmount;
    }
}
```

Configuration settings are acceptable in class properties because they:

* Do not change during the object's lifetime
* Define the service's base behavior
* Are part of the application's configuration
* Do not depend on the execution context

### 2. Dependencies

These are other services or resources needed for operation:

```php
class OrderProcessor
{
    public function __construct(
        private readonly OrderRepository $orderRepository,
        private readonly PaymentGateway $paymentGateway,
        private readonly LoggerInterface $logger
    ) {}
}
```

Dependencies can be stored in properties, as they:

* Are tools for performing operations
* Are usually stateless themselves
* Are injected via the constructor
* Do not change during operation

### 3. Ephemeral State

This is exactly what should be avoided - data that changes during operation:

```php
// Anti-pattern!
class BadOrderProcessor
{
    private ?User $currentUser = null;
    
    // Avoid methods like this!
    public function setUser(User $user): void
    {
        $this->currentUser = $user;
    }
}
```

## Design Recommendations

1. Configuration:
   * Pass via constructor
   * Make immutable (`readonly`)
   * Use value objects for complex configurations

2. Dependencies:
   * Inject via constructor
   * Use interfaces
   * Do not change after object creation

3. Ephemeral State:
   * Pass via method parameters
   * Avoid `set*` methods
   * Use DTOs to group parameters
   * Store in DB or cache if necessary

## Conclusion

Designing stateless services is not just a trendy architectural decision. It's a necessity in modern PHP development, where applications must be ready for scaling, parallel execution, and operation in long-running processes.

A proper understanding of the differences between configuration, dependencies, and ephemeral state allows for the creation of clean and maintainable services that are easy to test and safe to use in any context.

Remember: a service that handles a single request today might become part of a complex asynchronous process tomorrow. By designing it stateless from the beginning, you protect yourself from numerous potential problems in the future.
