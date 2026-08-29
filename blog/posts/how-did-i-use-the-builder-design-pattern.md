---
url: >-
  https://viktorprogger.github.io/blog/posts/how-did-i-use-the-builder-design-pattern.md
description: A story from my practice about the situation where a had to create a Builder
---
There are many architectural patterns in the development world. Some of them we use every day, some - less often. Surely each of you has seen the Singleton and the Factory many times. Many of you have written them on your own. But when I first read about [the Builder pattern](https://refactoring.guru/design-patterns/builder), at first I did not understand in what situation it can be applied. What is completely ridiculous: I regularly used its implementation (QueryBuilder from the Yii framework), but my eye was so blurry that I could not match the name and functionality of this class and the corresponding design pattern 😂 Of course, after a while it dawned on me. And over more time, a situation was found in which the Builder pattern fit perfectly.

At the moment this has happened only once. Usually I worked on adult projects, where either such problems had already been solved, or it was a terrible legacy without resources for refactoring. So, where did I need the Builder?

### Task: product feeds optimization

The project was an online store and a marketplace at the same time. A dozen commodity feeds were generated on it. These were files of various formats with a list of products, their categories and characteristics. And they were uploaded to other marketplaces or, more often, to various platforms that provide advertising and analytics services. Each feed had its own settings: what products/brands/categories/etc. should be included in it, what data should be available for products, etc. The settings were stored in the database and displayed in the UI by our marketers.

Each feed was generated independently of the others. Initially, it took up to 10 minutes for each feed and heavily loaded the system, because it read data from the database very actively. And when it was necessary to generate all the feeds, this procedure stretched for an hour and a half, during which the reading replica of the database worked at the limit.

### First iteration of optimization

First of all, I've done two things:

* I've used SphinxSearch to search for products and get some data on them. The initial version of feeds was written even before the introduction of the Sphinx, and it was difficult to introduce something new into it, so hands have reached it only now.
* I've made feed generation a single process. The data could be obtained once, and based on it, the necessary feeds could be built. The feed was divided into two things: a filter that chose the suitable products, and a template that created a file from these products. We took a product and passed it through all the feeds. If the filter passed it, the feed wrote information about this product into a file. And so we did for each product.

:::info
This approach brought the time of creating feeds into a constant: about 6 minutes regardless of their number. However, it could have been better: in some feeds, only 20-25% of the products were used, but even when they were launched separately, all products were still run through the filter.
:::

### Second iteration: The Builder

The second thing that came to my mind was to create my own QueryBuilder with top-level “queries”: `withBrands()`, `withCategories()`, `withPhotoType()`, etc. When it was necessary to generate several feeds at a time, a QueryBuilder instance was taken for each of them, and they were merged into one to select from the database only those products that are needed by the current set of feeds. The two goals have been achieved:

* The independence of the entities in the feed generation module from the database structure was preserved.
* Feed generation speed was significantly improved for sertan cases.

As a result the generation code itself changed only in one place: getting products from the repository changed from `getProducts(): iterable` to `getProducts(QueryBuilder …$queries): iterable`

***

Tell me how often it happened to you in your practice to write Builders? I mean your own builders, not the ActiveRecord QueryBuilder extension 😉 I'm waiting for your comments in [the blog telegram chat](https://t.me/viktorprogger_channel_ru) 👍 See you!
