---
url: 'https://viktorprogger.github.io/posts/how-to-start-develop-opensource.md'
description: >-
  A story about my way into OpenSource framework development and instructions
  for you how to do the same
---
We all use OpenSource products. Some of them we like.\
And for those we like, we often want to help to become even better. One way to help is financially. But we ourselves are developers, and sometimes we imagine ourselves among the authors of a wonderful tool that we use with pleasure. At least that's how it used to be for me. And then the dream became a reality.
If this is also a dream for you, you're welcome to the further reading: there is my story and the answer, how you can join to develop your favorite OpenSource tool too.

![](/opensource.jpg)

[**TL;DR: How To**](#how-to-start-writing-code-in-an-opensource-project-yourself)

::: note Thank you
Thanks [Leonid Chernenko](https://leonidchernenko.ru/) for help with this article proofreading.
It is my first public work made in English, and I'm very thankful for his help and support.
:::

### My story

Now I'm one of the Yii3 core development team. I like this framework very much and I like to help it to become better.
The story about how did I come into the core team is as simple as possible. Once I came to the framework telegram chat and saw Alexander Makarov, the framework maintainer, answering the questions in the chat. He was answering every question, not just "interesting" or "useful" ones. It seems that he wrote something on the topic of being written to him in a PM, if there are any questions for him personally, but I'm not sure. Frankly, I was dumbfounded. You can say that Alexander was my idol, and suddenly - here he is, in a chat room where you can chat with him just like with my friends. That moment I've understood I didn't talk to him before just because I've decided he won't want to talk to me. For the same reason I didn't participate in the life of the Yii framework.

Then I decided on an adventure. I was scared, but I have texted Alexander anyway. That time I still was thinking he is smthng more than me. But I convinced myself that, first of all, we are both people, which means that communication is not alien to us. At last, I've written smthng like that:

> Hi! My name is Viktor, I'm a PHP developer. I like the Yii framework very much, and I'd like to help in developing of the new, the third version of the framework. Tell me please, how can I start?

And he thanked me for my enthusiasm and gave me links to some GitHub tickets, which I soon got to work on. Tickets were really at the level of "where to start". I don't remember what exactly it was, but it was the little things. Gradually, I got involved in this process and began to do more and more complex things. For example, I wrote the first version of the `yiisoft/validator` library. Not completely, just proof of concept. At that time I changed my main job and for some months I could not do OpenSource development. But the Yii core team guys took my code, finalized it and turned it into a full-fledged library. Now it has completely different architectural approaches, but for me personally it means a lot: it was my first serious contribution to the popular and, IMHO, the best framework.

### My results of the OpenSource development

Of course, I got a lot of experience. I continue to happily engage in OpenSource in my free time and continue to get more and more new experiences. To single out the most striking acquisitions, I will name these points:

#### Deep understanding of SOLID principles and overall application and library architecture.

In commercial development, you usually don't have the opportunity to sit down and analyze various architectural solutions, write several versions and choose one of them, or even completely rework the original design with a deep refactoring. There are other values in OpenSource: while in commercial development one of the main criteria for choosing anything is the price/quality ratio, in the OpenSource the emphasis is always on quality. And now I use the experience of creating SOLID architectures, gained in OpenSource, in commercial development, where it is more important not to follow the principles to the end, but to understand where and how they should not be followed, what will be the price of their observance and violation.

#### Writing tests.

I had little experience writing tests before beginning the Yii3 development. Basically it were only the things I've learned myself. In YiiSoft we apply an integrated approach:

* [Psalm](https://psalm.dev), statical typification checker.
* [PHPUnit](https://phpunit.readthedocs.io/ru/latest/), a framework for unit tests.
* [Infection](https://infection.github.io/), a tool for checking unit tests quality (mutational testing).

It could mean nothing, but we do as full code coverage as possible, set Psalm level 1 and improve Infection MSI up to 100% for every package which is going to be released (see [yiisoft/assets](https://github.com/yiisoft/assets) for example). It's very valuable for me that I've come to YiiSoft on start of the new evolution iteration of the framework. These tools and the way of their usage were selected by the whole team. We were covering the first packages with tests together. By the way, I've covered with tests my first package so badly that lately I was terrified. It was very difficult to maintain and evolve package with such tests. But this is the case when you learn from mistakes. Although it is these tests that have not yet been refactored at the time of this writing  😆

#### Experience of working in a self-managing collective

Yes, we all are equal in the team. We usually make more sense for Aleksander Makarov's ([@samdark](https://github.com/samdark)) opinion. But he doesn't shy away from the others too. If anyone have questions or objections, we will discuss it till our common solution.

We've got our cons too. For example, for a long time it was Alexander who performed all managerial work. He was our boss in fact. He made most of architectural decisions, merged PRs and made releases. He also was the one who set the team's priorities.

Once he dropped out of the framework development for a couple of months. We had to get out ourselves. I'm glad to say: we did it. Not at once, but we changed our processes to avoid Alexander in most cases. We've phoned and discussed the most pressing issues, agreed on compromises and priorities. @Samdark came back in some time too. But he is not a bus-factor for the team since then. He is still the most respected team member and the framework maintainer. But now he is doing more in terms of helping the team to improve processes and discuss decisions instead of direct managing. *This is a wonderful experience: people united by one idea, on their own initiative, cooperated and are doing a common thing.* And most of us are doing this thing for free. You can get a similar experience in a "teal company", I think. But there are still a few such companies, so OpenSource is more simple way to get it.

#### Summary

The above points are the experience that turned out to be the most vivid for me. It will be smthng else for you. But the most important thing is that everyone can find in OpenSource a lot of experience, not available on commercial development. Not to mention how valuable the line that you have experience developing a popular OpenSource tool is on your CV.

### How to start writing code in an OpenSource project yourself

![](/tom_jerry_wait.jpg)

It doesn't matter which one of the OpenSource projects do you like the most. Its team will be glad to get your help. The good news is that it's pretty easy to start writing code for it. The necessary steps are:

1. Find out what help do they need. Find an actual issue list or ask in chat.
2. Write the corresponding code and get review for it.
3. Fix the review comments, if any.

That's all. After that hundreds/thousands/millions of people will use your code! All you need to continue is just to reiterate the above steps again.

The most difficult thing is to do the first step on this path. You should find out the way you can help. It is really pretty easy: the only thing you have to do is to ask a question in the tool community. I'm sure all the OpenSource tools have their own chats or forums for now. I.e. Yii3 has telegram chat rooms [@yii3en](https://t.me/yii3en) and [@yii3ru](https://t.me/yii3ru). The algorithm is simple: go into the chat, express your desire to help and ask them to submit a ticket that should be dealt with. If it's your first time, you'd rather all for smthng more simple to get into the development process before taking more complex things.

### What prevents you from joining an OpenSource product

The algorithm above is really outrageously simple, its implementation is available to everyone. But why do so few people do it?

#### 1. I have no time

You have a full-time job and a family, I guess. And you're busy for 24 hours in a day. Maybe even for 25 hours. However, the secret is simple: you can do OpenSource in your spare working time. It may seem that "spare" and "working" are incompatible things, but it is not so. The fact is that we are really productive only 2-4 hours out of the whole working day. The rest of the time is occupied by other things: code reviews, routine, bureaucracy, etc. But this is ideally, and usually all this time is occupied by procrastination. Consider: do your tasks really take as much time to complete as they do? Don't they? Then I propose an experiment: do your work for 2 hours a day with maximum immersion in it, without being distracted by anything. Prepare a clear plan of action, switch all chats and mail to silent, and go ahead. The second part of the plan is to be sure to dedicate part of your time to what brings pleasure, what you sincerely enjoy. Personally, I sometimes spend this time developing a framework :) Well, a significant part of the time - for sluggish work tasks that do not require much concentration. And do not forget about lunch, where you will not be doing work in any form for an hour. It is also desirable to arrange a walk or some kind of physical activity and exercises during breaks.\
I bet that with this mode, your efficiency will increase, and part of the freed up time can be given to your favorite OpenSource project.

#### 2. I'm inexperienced

So OpenSource is for you! This is an excellent place to gain experience. As a rule, there are enough tasks of very different complexity: not only to develop new architectural approaches and performance optimization, but also various simple but necessary little things: change the code style, add documentation, rename files and classes, etc.  And you will help a lot if you take these things upon yourself. At the same time, you are free to take more complex issues where you need to figure out for implementation, and even ask for help from the main developers of the product. Entering OpenSource development is possible at any stage of professional development if you already know well the programming language used in the product.

#### 3. I'm afraid I'll be laughed at

It was very strange for me to hear this reason. I have never met developers of OpenSource tools so toxic that they somehow ridicule those who want to help them. For what? The only thing that comes to mind is that your friends, having heard that you decided to make your contribution to OpenSource, out of envy and self-doubt, they may try to somehow insult you, stop you, so that in no case it turns out that you are "better than them".  Throw such acquaintances if you have them: you should not go under with them.  Our path is only forward!

#### 4. I'm scared

Of course, you are! Like anyone else who is going to start something new in his life. It's ok. There is no personal development without fear. A brave person is not one who is not afraid, but the one who acts in spite of fear. Therefore, the best decision is to be afraid and do it 😊 Find new colleagues, and who knows, maybe - friends and a mission?

***

Please, tell me about your OpenSource experience and wishes. Did you ever write some code for an OpenSource project?
Did you ever want to do that? Does anything stop you now? I'll be glad to talk to you in [the blog Telegram chatroom](https://t.me/viktorprogger_channel_ru).
