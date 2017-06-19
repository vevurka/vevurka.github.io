---
layout: single
title: Debugging kafka client in Python
date:   2017-05-14 23:31:02 +0200
categories: [dsp17, python, debug]
tags: [python, kafka, mistakes, debug-story]
excerpt: What to do if you want to stop kafka consumer properly?
---

Recently I had some issues with **Kafka** client in **Python**. I wanted to write
a **Kafka  event consumer**, which will be able to *stop gracefully*
on `SIGTERM` or `SIGINT` *signal*.
The other requirement is to be able to run *multiple* instances of this **consumer**.

I decided to use the newest **python-kafka** library - currently version **1.3.3**.
But maybe let's start from what actually is this **Kafka**.

## Some info on Kafka and python-kafka

Before we start here is an simple overview on **Kafka** and **python-kafka**
library. If you know any basics just skip this part.

### Very short overview on **Apache Kafka**

**Kafka** is a distributed streaming platform created by **Apache**.
It uses its own protocol called **kafka**.
It's quite similar to **publish-subscribe** model like in *MQTT* protocol, but also uses
**queue** concepts
like in *RabbitMQ* (*AMPQ* protocol). We can say that it's a **generalization** of
**publish-subscribe** and **queue** model.

In regular **queue**, event can go to only one *subscriber*, then
it's gone - in **Kafka** it will go to **all** *subscribers*. On the other hand
in **publish-subscribe** model, **scaling** is much harder. In **Kafka** they resolved
this issue with *scaling* somehow (I don't know yet how!).
As it supposed to be short, I'll write more about **Kafka** in future.

### Very short overview on **python-kafka**

As **Kafka** is using **publish-subscribe** model - client for it
needs an *event consumer* and an *event producer*.
Library **python-kafka** which is a *Python* client for **Kafka**, according to
*documentation* consist of:
* `kafka.KafkaConsumer` - it should **consume** Kafka *events* (so it's a **subscriber**)
* `kafka.KafkaProducer` - for **producing** Kafka *events*

Also there is `kafka.KafkaClient` which is used by both of them - it shouldn't
be used *explicitly*.

If you look to a *documentation* page or *old versions* of this library, there
are also: `kafka.SimpleConsumer` and `kafka.SimpleProducer` - they
are to be **deprecated** soon, so I'm not going to use them.

## Graceful exit

I wanted to write a **Kafka  event consumer**, which
will be able to *stop gracefully*
on `SIGTERM` or `SIGINT` *signal*. By *stop gracefully* I mean here
that I'll use `kafka.KafkaConsumer.close()` - it's *rightful* method for closing
**consumer**.
I had two approaches:

1. Using `kafka.KafkaConsumer` in **thread**
2. Use `kafka.KafkaConsumer` explicitly

### Thread approach

To be able to close my **consumer** I decided to start from
putting `kafka.KafkaConsumer` into a **thread**.
I created a `Consumer` class which was my **thread** class. It
was supposed to run `kafka.KafkaConsumer` in it:

{% highlight python %}
    import logging
    import signal
    import threading

    from kafka import KafkaConsumer

    logger = logging.getLogger(__name__)


    class Consumer(threading.Thread):
        def __init__(self, name):
            super(Consumer, self).__init__(name=name)
            topics = ['creator_topic']
            self.consumer = KafkaConsumer(*topics,
                                          bootstrap_servers=['localhost:9092'],
                                          group_id='dev')
            self.should_stop = False

        def run(self):
            for message in self.consumer:
                if self.should_stop:
                    self.consumer.close()
                    break
                logger.info(message)
                # Do sth about message
            self.consumer.close()

        def exit_gracefully(self, sig):
            logger.info("Closing %s because received %s", self.name, sig)
            self.should_stop = True


    def exit_gracefully(sig, threads):
        logger.info("Exiting %s", threads)
        for t in threads:
            t.exit_gracefully(sig)
            t.join()


    def main():
        consumer_threads = [Consumer('c1'), Consumer('c2')]
        signal.signal(signal.SIGINT,
                      lambda sig, _: exit_gracefully(sig, consumer_threads))
        signal.signal(signal.SIGTERM,
                      lambda sig, _: exit_gracefully(sig, consumer_threads))
        for consumer_thread in consumer_threads:
            consumer_thread.start()

{% endhighlight %}

After receiving `SIGINT` or `SIGTERM` all running **threads** were to be marked with
`should_stop` flag. With this flag set, loop in run for each **thread** was supposed
to **stop**. Everything was fine, except that they stopped only after receiving
next message. This is because `kafka.KafkaConsumer` is actually an **iterator**.
Anyway, when I was **debugging** I realized it's not a good approach at all, because
in `kafka.KafkaClient` class in file `client_async.py` there is a small comment:

    This class is not thread-safe!

I had to study almost all **consumer** related code of this library to find it. Also I could
read whole **documentation** to find it - it was quite hidden.
I'm not sure why they didn't stress it more.

So, I had to change my approach.

### Using kafka.KafkaConsumer explicitly

Still I needed some mechanism to be able to **stop** **iteration** of loop like this:

{% highlight python%}
     for message in self.consumer: # instance of kafka.KafkaConsumer
         logger.info(message)
         # Do sth about message
     self.consumer.close()
{% endhighlight %}

Because `kafka.KafkaConsumer` is deriving from `six.Iterator` (so Iterator compatible
with both **Python 2** and **Python 3**), it implements `__next__` method:

{% highlight python%}
    # kafka lib, group.py,

    class KafkaConsumer(six.Iterator):

    # ...

    def __next__(self):
        if not self._iterator:
            self._iterator = self._message_generator()

        self._set_consumer_timeout()
        try:
            return next(self._iterator)
        except StopIteration:
            self._iterator = None
            raise
{% endhighlight %}

It's clearly written that **iteration** (waiting for *next messages* mechanism) will
*stop* when `StopIteration` **exception** will be thrown. So I changed my
previous code a bit.
My `Consumer` now derives from `kafka.KafkaConsumer` (so it's not a thread anymore!)
and has methods `run` and `exit_gracefully`. Method `run` runs a **message iterator**,
method `exit_gracefully` throws `StopIteration` **exception** to finish
**message iterator** loop:

{% highlight python %}
    import logging
    import signal

    from kafka import KafkaConsumer

    logger = logging.getLogger(__name__)


    class Consumer(KafkaConsumer):

        def __init__(self, topics, name):
            super(Consumer, self).__init__(*topics,
                                           bootstrap_servers=['localhost:9092'],
                                           group_id='dev')
            self.name = name

        def run(self):
            for message in self:
                logger.info(message)
            logger.info("Iteration stopped in %s", self.name)
            self.close()

        def exit_gracefully(self, sig):
            logger.info("Exiting %s gracefully because got signal %s.",
                        self.name, sig)
            raise StopIteration("Closing")

    def main():
        topics = ['creator_topic']
        consumer = Consumer(topics, 'c1')

        signal.signal(signal.SIGINT,
                      lambda sig, _: consumer.exit_graceully(sig))
        signal.signal(signal.SIGTERM,
                      lambda sig, _: consumer.exit_gracefully(sig))
        consumer.run()

{% endhighlight %}

When receiving signal `SIGTERM` or `SIGINT` `exit_gracefully` is run. It raises
`StopIteration` which stops an iterator loop, after loop there is `self.close()`
instruction, which closes **kafka consumer** properly.


#### What about multiple consumers?

Solution above doesn't work as expected when having **more than one consumer**.
Let's say we want to close **consumers** on received `SIGTERM` or `SIGINT`
signal in loop like this:

{% highlight python %}
    def close_all(sig, consumers):
        for c in consumers:
            c.exit_gracefully(sig)


    def main():
        topics = ['creator_topic']
        consumer1 = Consumer(topics, 'c1')
        consumer2 = Consumer(topics, 'c2')

        consumers = [consumer1, consumer2]
        signal.signal(signal.SIGINT,
                      lambda sig, _: close_all(sig, consumers))
        signal.signal(signal.SIGTERM,
                      lambda sig, _: close_all(sig, consumers))
        for c in consumers:
            c.run()
{% endhighlight %}

It's impossible, because raised `StopIteration` exception
is **propagated** in code of `__next__` in `kafka.KafkaConsumer` -
I need to **interrupt** (send `SIGINT`)
as many times as I've **consumers** to stop this program. I've no idea how to
solve this... Any ideas?

### Some thoughts

They say we should always **close** everything *gracefully*:
close *sockets*, close *connections*, close *files*, etc. But sometimes it's not so
easy to do this properly. I made so big effort, spent so much time, to just
run `kafka.KafkaConsumer.close()`
method and still I'm not so successful... I've a partial success, because
I'm able to close all **consumers**, but still I've to send a few **interrupt** signals
instead of one.