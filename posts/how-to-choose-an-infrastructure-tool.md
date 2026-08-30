---
url: 'https://viktorprogger.github.io/posts/how-to-choose-an-infrastructure-tool.md'
description: How to make the best choice between different infrastructure tools
---
When an IT product evolves, there always comes a time when it becomes necessary to bring some new tool into it or replace the old one with something more suitable. While it seems that a lot of articles have been written on the topic of choosing tools, and there is nothing new to say, the question still remains relevant, in some places even sharp.

Here are the most popular approaches:

* Find something hype on the left road
* Take something proven and reliable on the right road
* Go straight ahead and use your favorite tool

They are good because you don't have to think when using them. You just take what you want - and that's it :) Well, if there is a need or a desire to approach the issue more seriously - welcome under the cut, we'll figure it out.

So, if not “quickly”, how to choose then?

### 1. Make a list of requirements

The very first and most obvious step. But not everything is as simple as we would like. The list should be relevant right now and for the foreseeable future. That is, you need to strike a balance between two directions:

* Do not look into the distant future with the potential growth of the company to a “MAANG killer” and increase RPS from the current 0.1 to 1000.
* Do not leave behind real requirements, current ones and from the foreseeable future (say, in 1-3 years).

### 2. Choose several candidates that meet all the requirements

And when this is done - conduct a comparative analysis of them, using the requirements drawn up earlier. Do not forget to take into account a few more points:

1. Ease of use.\
   The tool that you are going to use regularly should not interfere with your use of it, and even better, it should make this process pleasant and convenient.
2. Frequency of updates.\
   Some old, reliable tools eventually stagnate and stop being updated. In this case, even if such a tool seems to be the ideal solution now, there is a rather high probability that it will have to be changed again soon, as it will either no longer meet the security requirements, or will not implement the new features that its rapidly developing competitors will have. There are young tools on the other hand. It is possible to stagnate for them when there is too small community and a few maintainers.
3. The presence of a community, integrations, plugins.\
   The lack of a large community for a new cool tool and a large number of integrations that its more proven counterparts have can be a very serious obstacle in the development of your company. Not being able to quickly embed the tool into your current processes is a tangible problem. With older, slower, less user-friendly tools, you can often save yourself dozens or even hundreds of hours of work simply because your problems have already been solved by someone else. And no matter how cool a tool is, at the dawn of its development there is always one obstacle: users need to solve their problems on their own.
4. The presence of expertise in the team.\
   Even if an unfamiliar tool does not look complicated, and installation and configuration guides fit on one screen, this does not mean that you will not have difficulties in a month. And since you have no expertise in working with this tool, it is not known how long it will take you to eliminate errors or optimize its work.

In the general case, the analysis should take into account only the main functionality common to all systems. I'm talking about the fact that all infrastructure tools, without exception, sooner or later become obsolete. Immediately think about how you will change this tool to some other one. Not to some specific, but to an abstract one. Caching system - to another caching system. In the case of systems for which there are no standards like PSR, for this you will have to conduct a deeper analysis yourself to identify this common functionality.

::: tip  
If we are talking about a caching system, then you can be guided by the current PSR. At the same time, the caching system itself can provide much more features, as, for example, Redis does. I do not recommend using these features in code directly. Of course, Redis can be used not only as a caching system, but also as a database. This is fine. But using it as a caching system that uses database functions that are not provided by PSR will sooner or later lead to problems.
:::

### 3. Weigh all the pros and cons of the chosen instruments

Even in similar situations in similar companies, the choice can be radically different, since the value of one or another point will also change. For some, the ease of use of the system is more important, for some it is the size of the community and related tools, and for some this tool will be built into such a fast-growing business that bandwidth will become the main factor for it, and not at all ease of use and speed its embedding. You can create a table with weights for each tool feature, if you want to make this comparison more visual.

### Real Life Example

Now let's try to choose some real tool, which we will select according to the scheme above. The company where I work recently faced the need to change the queue server. Before the replacement, Redis was used, but it ceased to suit us. The volume of information passed through it is growing, its value is high, and we are not ready to lose it in the event of a failure. This implies the *main criterion by which it does not suit us: the reliability of message delivery*. We do not deal with routing messages between queues, the complex logic of their delivery, and other similar things. The growth of the traffic passed through the queues increases by a maximum of two times per year. Another criterion: the speed of launching a new message broker in production. We want to finish embedding in a month. So, some completely new tool is not suitable, because we are not ready to spend extra time on its embedding because of the lack of finished cases.

At the stage of choosing a tool, everything came down to a classic confrontation:

* **RabbitMQ**. A couple of our developers already had experience with it. It is also a fairly popular queue server that has a rich set of functionality and works according to the amqp standard. It is the most well documented and easier to install and maintain than other brokers that implement amqp.
* **Apache Kafka** - very popular among enterprise-level companies. Works according to its own standard.
* **NATS** was considered as one of the options for a long time, because it did not immediately become clear that it does not persist messages, which means that it loses them after it is turned off. And it was also against him that it only receives messages via UDP, which adds risks of their loss.

::: info Disclaimer
A detailed analysis of Kafka and RabbitMQ is beyond the scope of this article, there are plenty of similar materials on the Internet. I will focus solely on the choice according to our own criteria.
:::

So, let's evaluate the two remaining competitors in terms of their capabilities and shortcomings.

#### RabbitMQ

* Some people on our team have already worked with it, so it has a head start.
* There is a possibility of flexible message routing, thanks to the amqp protocol. The chip is cool, but it is not taken into account in the rating, because this cannot be reused when switching to another broker. Their use will make it more difficult to switch to another instrument when the moment is right.
* Method of clustering. RabbitMQ uses database synchronization for clustering and high availability. It works quickly, clearly, but does not forgive network errors. In other words, if the broker's nodes are located on different servers, in different data centers, network problems will inevitably arise, which will lead to data desynchronization in the nodes.
  ::: info Shovel
  There is also an alternative in the form of the Shovel plugin, which uses the amqp standard under the hood and simply forwards messages from the queue of one server to the exchanger of another, while correctly handling network errors, but adding significant overhead, because this is not just a copy of raw data, but a complete processing of a message.
  :::
* Message delivery guarantees depend on the settings and only two types are supported:
  * at least once, any message will be delivered one or more times
  * at most once, any message will not be delivered more then once (but it also may be not delivered at all)
* It is possible to enable writing messages to disk so that they are not lost when the queue broker is restarted.
* Rabbit's throughput is not bad, but it seriously drops when writing messages to disk is turned on, down to several thousand messages per second. Other brokers have orders of magnitude higher throughput (from hundreds of thousands to millions per second), but we are within these numbers in the foreseeable future.

#### Apache Kafka

* We've heard a lot about it, but haven't seen it live.
* Implements only pub/sub, does not know how to route messages. Which, again, is not a minus, but a statement of fact.
* Ability to cluster - available out of the box, without a noticeable loss of performance and restrictions on the local network, which allows you to make readonly replicas, store different queue shards on different servers, etc. Minus: another tool is required, ZooKeeper.
* Message delivery guarantees also depend on the settings, but all three possible types are supported, including “exactly once”, a guarantee that a message will be delivered to the recipient exactly once. However, with this mode of delivery guarantees, there are difficulties when setting up Kafka clusters. Of course, the corresponding recipes are already on the Internet.
* Writing messages to disk is not disabled due to architectural features: queues in Kafka are logs where messages are added by producers and from where they are read by consumers. In other words, all messages received by Kafka are always available for reading, including those that have been processed a long time ago.
* Bandwidth - about 1 million messages per second.

When comparing the pros and cons of these two tools, it becomes clear that Kafka wins tangibly. *And yet, we made a choice in favor of RabbitMQ.* It's all about its very first plus: this system is already familiar to us, and it is less likely that we will encounter problems in production. At the same time, we do not plan to make a geographically distributed system, i.e. features of Rabbit clustering in our case are not a minus.

In Kafka, however, its architectural difference brings significant difficulties. First, Kafka does not actually have a queue: he has logs divided into partitions, and exactly one consumer can read from each partition. We have never worked with such a scheme, and there is a high probability of a shot in the foot when working with real data in production. We especially feel apprehensive at the thought of scaling when processing messages. You can simply add consumers to a Rabbit. And with Kafka, it is necessary to think in advance about the number of partitions and the distribution of messages over them in order to preserve the order of tasks in some cases.

Our company is growing and developing, and we do not renounce the possibility that we will change RabbitMQ to Kafka in a couple of years, if suddenly our requirements for the queue broker change. And in order for this replacement to be painless, we will not get hung up on the features of the current tool.

### Conclusions

If you follow the above plan, you can choose any tool from the available options. Of course, it leaves room for risk: if you choose unequivocally the best available, but unfamiliar tool, you can stumble upon pitfalls during operation. These risks should also be taken into account when choosing a tool. I will give a plan for preparing for the choice of an infrastructure tool again:

1. Make a list of requirements, not missing the real ones and not inventing something beyond what is necessary.
2. Select several candidates and conduct their comparative analysis.
3. Evaluate winning and losing positions for different instruments, guided by current priorities.

Tell me, have you followed a similar plan in your practice? What oddities happened due to unaccounted for requirements or features of the tool? I’m waiting for you [in the telegram chat](https://t.me/viktorprogger_channel_ru), I’ll be glad to hear your stories and to chat with you!
