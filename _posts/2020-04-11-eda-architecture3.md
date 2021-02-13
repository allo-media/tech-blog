---
layout: post
title:  "An event driven architecture — part 3"
excerpt: "Implementation overview."
date:   2020-04-11 12:53:00 +0100
categories: architecture
tags: services broker event bus rabbitmq architecture
---

In this new post, let talk about the actual implementation of the event driven architecture.

## Topology

The topology describes how the bus is implemented on the message broker.

As message broker, we chose [RabbitMQ](https://www.rabbitmq.com/) for its reliability record and ease of use.

In RabbitMQ, the topology is set up by instantiating *exchanges* and *queues*.

Exchanges are kind of routers, and queues are bound — using subscriptions to particular *routing keys* — to them by client applications to store their messages waiting for processing. It is a good practice to consider the queues as private to the logical service (but are shared by the workers of that logical service). We use the message type name as routing key.

The topology is simple and made of three exchanges:

 - a reliable *events* exchange with:
    - persistent consumer queues (survive broker and service restarts)
    - persistent messages (survive broker and service restarts)
    - message processing acknowledgment by clients
    - message confirmation by broker (for early detection of network transient failures)
    - *topic* routing (to allow wildcard monitoring)

  - a reliable *commands* exchange with:
    - persistent consumer queues (survive broker and service restarts)
    - persistent messages (survive broker and service restarts)
    - message processing acknowledgment by clients
    - message confirmation by broker (for early detection of network transient failures)
    - *topic* routing

 - a simple *logs* exchange with:
    - no persistence (to avoid over-flooding the broker memory in case nobody is consuming the logs)
    - no message acknowledgment or confirmation (for speed)

The Result of a Command is sent to the `commands` exchange too.

Instances of a same logical service are called *workers* of that service, and they share the same queue. The broker guarantees that a message is processed by one and only one worker of the pool attached to the queue.

The message processing acknowledgment by clients is a very useful mechanism to ensure no data is lost — that is, a message is guaranteed to be processed at least once — and to allow efficient load balancing. Indeed, the message broker won't remove a message from the queue until the worker who took it for processing tells it that it has finished its job. During this time, the message is reserved. If the worker who took the message goes down before acknowledging, the message becomes available for any other worker, or the same worker when it comes back. Moreover, the broker knows at any given time which workers are busy and which ones are idle, so it can better share the load between them.

Note that this is the base topology. As the topology is **created by the services themselves** instead of a central configuration, they can extend it locally (i.e. on their side) for their own needs.
That also means they must all agree on the base topology exposed above to join the bus.

To avoid code duplication, we developed a python mini-framework that implements this base topology and all the desired behaviors of the services. We'll talk about it in the next post: *Framework and tools*!
