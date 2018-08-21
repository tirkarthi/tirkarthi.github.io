---
layout: post
title:  Redis streams and asyncio
date:   2018-08-18 00:00:29 +0530
categories: programming
---

**This is a python translation of my tutorial in Clojure. You can check the clojure one [here](https://tirkarthi.github.io/programming/2018/08/17/redis-streams-clojure.html)**

Stream processing is a common task in analytics related applications where large amount of data like click stream data are ingested into the system that has to processed by some worker. [Kafka](https://kafka.apache.org/) is a common choice here when reading from streams and also to coordinate message delivery between different workers. But it comes at an operational cost where there are a lot of components like [ZooKeeper](https://zookeeper.apache.org/) are involved that you have to maintain. [Amazon's kinesis](https://aws.amazon.com/kinesis/) is also a managed service from AWS to handle large amount of data.

For a lot of years Redis has also been modelled in a similar fashion with pub/sub and having blocking lists but they are not entirely designed with stream processing as the goal. They don't offer persistence in case of pub/sub where it's fire and forget method. Blocking list are one way but we have to co-ordinate between different workers so that two different consumers get the same message and so on. With this in mind antirez, creator of Redis came up with [disque](https://github.com/antirez/disque) from the ground up. Then a lot of the ideas were added back to Redis in the form of streams. It's an entirely new data structure focused on stream processing. It's in beta now and will be available with [Redis 5.0](https://redis.io/download) . You can download the beta for this tutorial.

### Requirements

This tutorial uses aioredis as the Redis client and streams support is present in master but it's not yet released. We also need the beta version of redis-server and redis-cli since streams are not GA as of now. You can clone the repo and create a virtualenv to install the dependencies. Streams are still in beta so you can either download one of the beta release binaries from https:or compile from source on the unstable branch from GitHub. Code for the tutorial is at [GitHub](https://github.com/tirkarthi/python-redis-streams-playground)

### Read and write on streams

Now that we have Redis setup we can try to insert some data. Redis provides `XADD` through which we can specify the stream name and then specify a set of key-value pairs for the given stream which is added as the data. In addition to that we also need to specify a pointer for the data. A sample to insert temperature for chennai city as stream name will be below :

```
127.0.0.1:6379> XADD chennai * temperature 26 humidity 10
1534572307702-0
```

Aioredis

```python
import asyncio
import aioredis

loop = asyncio.get_event_loop()

async def main():
    stream = 'chennai'
    fields = {b'temperature': b'26', b'humidity': b'10'}

    redis = await aioredis.create_redis('redis://localhost', loop=loop)
    result = await redis.xadd(stream, fields)
    print(result)
    redis.close()
    await redis.wait_closed()

loop.run_until_complete(main())

# b'1534866436902-0'

```

Note that the returned value might be different on your computer since it's based on the current time at which the command is executed. Now we have created a stream named chennai and inserted a record of temperature and humidity. Redis autogenerates this value but we can use our own value which needs to be an integer and this has to be in an increasing order for every command. We also cannot insert two different records for the same timestamp and hence we have "-0" at the end. In case we need to insert it for the same timestamp value we need to specify the timestamp value as "1534866436902-1" and so on. They are immutable and increasing in order that we cannot change the value or go back in time to insert something in the middle of the stream. We can think of it as an append-only log.

Now that we have inserted a record we can read it back with `XRANGE` command as below by specifying the stream name and also the start and end which in this case is - and + to read the entire stream we can also specify an optional count of records to be returned :

```
127.0.0.1:6379> xrange chennai - + count 1
1) 1) 1534866436902-0
   2) 1) "temperature"
      2) "26"
      3) "humidity"
      4) "10"
```

aioredis

```python
import asyncio
import aioredis

loop = asyncio.get_event_loop()

async def main():
    stream = 'chennai'
    redis = await aioredis.create_redis('redis://localhost', loop=loop)
    result = await redis.xrange(stream)
    print(result)
    redis.close()
    await redis.wait_closed()

loop.run_until_complete(main())

[(b'1534866436902-0', OrderedDict([(b'temperature', b'28'), (b'humidity', b'10')]))]
```

Here instead of using - and + we can specify two different timestamps to read records only between those intervals which is useful to process timeseries data.

```
127.0.0.1:6379> XADD chennai * temperature 26 humidity 10
1534866659896-0
127.0.0.1:6379> XADD chennai * temperature 26 humidity 10
1534866664116-0
127.0.0.1:6379> XADD chennai * temperature 26 humidity 10
1534866668433-0
127.0.0.1:6379> xrange chennai 1534866659896-0 1534866668433-0
1) 1) 1534866659896-0
   2) 1) "temperature"
      2) "26"
      3) "humidity"
      4) "10"
2) 1) 1534866664116-0
   2) 1) "temperature"
      2) "26"
      3) "humidity"
      4) "10"
3) 1) 1534866668433-0
   2) 1) "temperature"
      2) "26"
      3) "humidity"
      4) "10"
```

```python
import asyncio
import aioredis

loop = asyncio.get_event_loop()

async def main():
    stream = 'chennai'
    redis = await aioredis.create_redis('redis://localhost', loop=loop)
    result = await redis.xrange(stream, start=b'1534866659896-0', stop=b'1534866668433-0')
    print(result)
    redis.close()
    await redis.wait_closed()

loop.run_until_complete(main())

[(b'1534866659896-0', OrderedDict([(b'temperature', b'26'), (b'humidity', b'10')])),
(b'1534866664116-0', OrderedDict([(b'temperature', b'26'), (b'humidity', b'10')])),
(b'1534866668433-0', OrderedDict([(b'temperature', b'26'), (b'humidity', b'10')]))]
```

Redis also provides XREVRANGE through which you can read it in reverse with similar options

```python
import asyncio
import aioredis

loop = asyncio.get_event_loop()

async def main():
    stream = 'chennai'
    redis = await aioredis.create_redis('redis://localhost', loop=loop)
    result = await redis.xrevrange(stream, start=b'1534866668433-0', stop=b'1534866659896-0')
    print(result)
    redis.close()
    await redis.wait_closed()

loop.run_until_complete(main())

[(b'1534866668433-0', OrderedDict([(b'temperature', b'26'), (b'humidity', b'10')])),
(b'1534866664116-0', OrderedDict([(b'temperature', b'26'), (b'humidity', b'10')])),
(b'1534866659896-0', OrderedDict([(b'temperature', b'26'), (b'humidity', b'10')]))]
```

### Producer and consumer

So now we can ingest some data and read from it but we have to record the last read value every time so that we can use it in the next call to continue from there and doing XRANGE is like polling the stream for every new record keeping track of the things we have read. Redis provides a blocking API called XREAD through which we can listen for new records in the stream instead of keeping track of the last position for each run.

Here we specify the block time in milliseconds as 0 which blocks indefinitely and we can pass the stream name and the position after which we need to wait for messages. "$" is the default that says XREAD should use as last ID the maximum ID already stored in the stream mystream, so that we will receive only new messages, starting from the time we started listening.

```python
import asyncio
import aioredis

loop = asyncio.get_event_loop()

async def main():
    stream = b'chennai'
    redis = await aioredis.create_redis('redis://localhost', loop=loop)
    result = await redis.xread([stream])
    print(result)
    redis.close()
    await redis.wait_closed()

loop.run_until_complete(main())

[(b'chennai', b'1534867553288-0', OrderedDict([(b'temperature', b'26'), (b'humidity', b'10')]))]
```

Now we can open redis-cli to add some new data that makes it to be received in the REPL

```
127.0.0.1:6379> XADD chennai * temperature 26 humidity 10
1534867553288-0
```

If we have specified a block of 1000ms then it will timeout after a second if there is no data and return `[]`

```python
import asyncio
import aioredis

loop = asyncio.get_event_loop()

async def main():
    stream = b'chennai'
    redis = await aioredis.create_redis('redis://localhost', loop=loop)
    result = await redis.xread([stream], timeout=1000)
    print(result)
    redis.close()
    await redis.wait_closed()

loop.run_until_complete(main())

[]
```

On a similar note we can listen from various streams with XREAD command and lit alt channels on core.async and it returns the channel name along with the data received. For example the below listens for chennai and coimbatore weather data and in the redis-cli when we give data for coimbatore it returns that particular data.

```
127.0.0.1:6379> xadd coimbatore * temperature 21 humidity 10
1534868080478-0
```

```python
import asyncio
import aioredis

loop = asyncio.get_event_loop()

async def main():
    streams = ['chennai', 'coimbatore']
    redis = await aioredis.create_redis('redis://localhost', loop=loop)
    result = await redis.xread(streams)
    print(result)
    redis.close()
    await redis.wait_closed()

loop.run_until_complete(main())

[(b'coimbatore', b'1534868080478-0', OrderedDict([(b'temperature', b'21'), (b'humidity', b'10')]))]
```

Note that the messages are not lost though you can still access them with XRANGE and also use XREAD with different positions.

### Consumer groups

Now that we have different producer and consumers present for each stream we can put XREAD command in a while loop and make sure it receives new messages for the given streams but there comes a point where the producer produces a lot of data and the consumer takes up some processing time where we can't handle the new with a single consumer so we need to introduce parallelise and introduce multiple consumers but we also need to handle other cases like balancing the workload, making sure the message is processed properly by the given consumer and it's not delivered to two consumers at the same time leading to double processing.

Redis gives us the ability take care of these with the concept of XGROUP where we group different streams under a given group and then Redis takes care of balancing the workload, keeping track of message delivery and so on. First we need to create a group with one or more streams attached to the group as below

```
127.0.0.1:6379> XGROUP CREATE chennai mygroup $
OK
127.0.0.1:6379> XGROUP CREATE coimbatore mygroup $
OK
```

We have specified "$" at the end which means that the group will listen for messages created after the group is created. We can now create a consumer called alice for the group and to listen for stream chennai. We can also make alice listen for both streams that are associated with the group.


Since we have delivered one message to alice we can spin up another consumer called bob in another REPL and listen in the same way. But let's also keep alice waiting. So when we deliver a message with both alice and bob listening but alice with a message that has been received it's to bob. Then if we tried a third message since bob has a message it's delivered to alice. So we can see it's delivered in a uniform manner across the consumers. Now if we stop bob from running then we can see all messages only delivered to alice

Now in two different shells start the below with alice and bob as first argument and enter the below commands

```
127.0.0.1:6379> xadd chennai * temperature 21 humidity 10 # delivered to alice
1534868318343-0
127.0.0.1:6379> xadd chennai * temperature 29 humidity 10 # delivered to bob
1534868336068-0
127.0.0.1:6379> xadd chennai * temperature 29 humidity 10 # delivered to bob
1534575541148-0
127.0.0.1:6379> xadd chennai * temperature 29 humidity 10 # delivered to alice
1534868389767-0
127.0.0.1:6379> xadd chennai * temperature 29 humidity 10 # Stop bob. Delivered to alice since bob has stopped listening
1534868390543-0
```

```
$ python consumer.py alice
Consumer group already exists
processing [(b'chennai', b'1534868318343-0', OrderedDict([(b'temperature', b'21'), (b'humidity', b'10')]))]
processing [(b'chennai', b'1534868389767-0', OrderedDict([(b'temperature', b'29'), (b'humidity', b'10')]))]
processing [(b'chennai', b'1534868390543-0', OrderedDict([(b'temperature', b'29'), (b'humidity', b'10')]))]
```

```
python consumer.py bob
Consumer group already exists
processing [(b'chennai', b'1534868336068-0', OrderedDict([(b'temperature', b'29'), (b'humidity', b'10')]))]
```

For each messages received by the consumer we need to acknowledge that the message has been processed with XACK. If we don't acknowledge it then it will be marked as pending by Redis. Redis also provides us XPENDING through which we can see the pending messages. We have 5 pending messages for chennai and we have alice with 4 and bob with 1 with 1534868318343-0 as minimum position and 1534868390543-0 as maximum position for the pending messages.

```
127.0.0.1:6379> XPENDING chennai mygroup
1) (integer) 4
2) 1534868318343-0
3) 1534868390543-0
4) 1) 1) "alice"
      2) "3"
   2) 1) "bob"
      2) "1"
```

We can also see the pending messages and the number of milliseconds they have been in pending state with each consumer.

```
127.0.0.1:6379> xpending chennai mygroup - + 10
1) 1) 1534868318343-0
   2) "alice"
   3) (integer) 250902
   4) (integer) 1
2) 1) 1534868336068-0
   2) "bob"
   3) (integer) 233177
   4) (integer) 1
3) 1) 1534868389767-0
   2) "alice"
   3) (integer) 179478
   4) (integer) 1
4) 1) 1534868390543-0
   2) "alice"
   3) (integer) 178476
   4) (integer) 1
```

We can acknowledge the message ID from redis-cli since XACK or from Python

```
127.0.0.1:6379> XACK chennai mygroup 1534868389767-0
(integer) 1
```

```
127.0.0.1:6379> xpending chennai mygroup - + 10
1) 1) 1534868318343-0
   2) "alice"
   3) (integer) 304068
   4) (integer) 1
2) 1) 1534868336068-0
   2) "bob"
   3) (integer) 286343
   4) (integer) 1
3) 1) 1534868390543-0
   2) "alice"
   3) (integer) 231642
   4) (integer) 1
```

The above info can be obtained as below :

```
import asyncio
import aioredis

loop = asyncio.get_event_loop()


async def main():
    streams = 'chennai'
    redis = await aioredis.create_redis('redis://localhost', loop=loop)
    result = await redis.xpending(streams, 'mygroup')
    print(result)
    redis.close()
    await redis.wait_closed()

loop.run_until_complete(main())

[3, b'1534868318343-0', b'1534868390543-0', [[b'alice', b'2'], [b'bob', b'1']]]
```

### Weather processing

Now we can write a consumer that pushes data for every second with temperature and humidity without any delay that is around 86400 messages. We can spin up a worker called that sleeps for 1 sec to print the message. Then we can spin up bob as another worker and we can see faster processing. Then we can stop bob and alice returns back to taking up all the messages. Thus Redis helps in distribution of the load and making sure the messages are delivered to the consumer only once. We can also see it takes only around 3-4MB to have 86400 messages in a Redis instance. First we need to flush everything and create stream with some dummy data and then create a group for the city. Now when we start the produce all the messages will be up for consumption from the point of view of the group.

```
127.0.0.1:6379> FLUSHALL
OK
127.0.0.1:6379> xadd chennai 1 temperature 29 humidity 10
1-0
127.0.0.1:6379> XGROUP CREATE chennai mygroup $
```

Alice

```shell
$ python consumer.py alice
processing [(b'chennai', b'1534868871014-0', OrderedDict([(b'temperature', b'28'), (b'humidity', b'18')]))]
processing [(b'chennai', b'1534868871015-0', OrderedDict([(b'temperature', b'23'), (b'humidity', b'17')]))]
processing [(b'chennai', b'1534868871015-2', OrderedDict([(b'temperature', b'21'), (b'humidity', b'17')]))]
```

** Bob ** : Bob receives the alternate messages as below with the load shared across the consumers

```
$ python consumer.py bob
Consumer group already exists
processing [(b'chennai', b'1534868871014-1', OrderedDict([(b'temperature', b'21'), (b'humidity', b'10')]))]
processing [(b'chennai', b'1534868871015-1', OrderedDict([(b'temperature', b'23'), (b'humidity', b'11')]))]
processing [(b'chennai', b'1534868871015-3', OrderedDict([(b'temperature', b'22'), (b'humidity', b'15')]))]
```

Default event loop

```shell
time python producer.py
Inserting 86000 records took 13.727497816085815 seconds
python producer.py  8.68s user 1.61s system 71% cpu 14.438 total
```

uvloop as event loop is faster here compared to the default one

```shell
$ time python producer.py
Inserting 86000 records took 9.74830675125122 seconds
python producer.py  5.24s user 1.31s system 65% cpu 10.002 total
```

Clojure takes around 2 seconds for same amount of inserts. I don't know what is the issue here which I need to profile.

**Memory**

```
127.0.0.1:6379> info memory
used_memory:2405472
used_memory_human:2.29M
used_memory_rss:2351104
used_memory_rss_human:2.24M
used_memory_peak:5161552
used_memory_peak_human:4.92M
used_memory_peak_perc:46.60%
used_memory_overhead:1034550
used_memory_startup:984800
used_memory_dataset:1370922
used_memory_dataset_perc:96.50%
...
127.0.0.1:6379> xlen chennai
(integer) 86400
```

### Helpful links

* [streams introduction](https://redis.io/topics/streams-intro)
* [Talk by Antirez with live demo](https://www.youtube.com/watch?v=qXEyuUxQXZM)
* [Live demo but little older](https://www.youtube.com/watch?v=ELDzy9lCFHQ)


### Conclusion

Streams are pretty great and if you already have Redis in your tech stack then I think they will be a great fit for stream processing instead of adding more components and also easy to monitor and administer. They are still on beta but will be GA soon and I love to use them in production. Thanks much Salvatore Sanfilippo and the community for a great software that is known for its stability and code quality.

_Opens maggie for dinner_

If you like this post then you might like my other posts on similar subjects :

* [Analysing 10k modules in Clojure ecosystem for JDK 9 problems](https://tirkarthi.github.io/clojure/2018/03/17/data-mining-clojars.html)
* [Improving Python's error messages with type annotations](https://tirkarthi.github.io/programming/2018/07/28/type-annotations-error-messages.html)
* [Analysing Rust crates for top dependencies and security issues](https://tirkarthi.github.io/rust/2018/03/30/analyzing-crates-data.html)
* [Analysing 30k issues CPython bug tracker](https://tirkarthi.github.io/python/2018/06/26/analyzing-python-bug-tracker.html)
