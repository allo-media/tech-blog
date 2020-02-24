---
layout: post
title:  "An event driven architecture — part 4"
excerpt: "Framework and tools."
date:   2020-04-18 12:53:00 +0100
categories: architecture
tags: python services rust broker event bus rabbitmq architecture graylog
---

## Eventail

We released it under a MIT license as [Eventail](https://github.com/allo-media/eventail). We are also working on a mini-framework for Rust.

## Monitoring

Beside the "classical" network and server monitoring solutions, we use more high level, specific tools for monitoring our event driven architecture.

For a global bus health monitoring, we use the [RabbitMQ management plugin](https://www.rabbitmq.com/management.html). It provide a live view of what's happening on the bus:
* message throughput;
* queue states;
* broker resource usage;
* client connection states…


As we said earlier, the services send their application logs on the bus, on a dedicated exchange. The [Eventail](https://github.com/allo-media/eventail) framework formats the log messages in the GELF format used by [Graylog](https://www.graylog.org/) so we can leverage that log management solution which can natively connect to [AMQP](https://www.rabbitmq.com/protocol.html) brokers like RabbitMQ. We can then use Graylog to define alerts, create graphical dashboard on the business processes, search the logs for debugging (by conversation identifier for example), etc…

In addition to application logs, the services also send periodical health checks on the same exchange, and those are also collected in Graylog: at any moment we know which services or workers are down, or are running in a degraded state.

## Testing, debugging and "repair"

The Eventail framework provides command line utilities for testing, debugging and replaying messages in the dead letter queue.

The testing commands allow the developer to publish events on the bus, or to send a command to a service and display the result on the console.

The debugging commands allow the developer to target and display logs on the console, or to subscribe to events and commands in order to display their content as they occur. There is also a command to inspect the content of a queue (without consuming its messages).

### Dead letters

We use [RabbitMQ policies](https://www.rabbitmq.com/dlx.html) for that.
Services just have to refuse a message and flag it in order to let the broker route it to the dead letter exchange. The framework manages this automatically in case of an unexpected exception is raised in the user code.

There should be something listening to the dead letters though, for this device to be useful.
As we saw, if a bug happens in a service, the message that raised the error is sent to the dead letter queue. There we can inspect its content with the inspection tool. We can also save it in order to "play" it on the developer computer  with the `publish_event` or `send_command` utilities. Once the bug is fixed in the service and is deployed in production, we can replay all the messages in the dead letter queue with the `resurrect` command. Thus, no data is lost.

## Outcome

> What do we get?

After four months in production, our event driven architecture brought us:

* Scaling — just start a new instance of a logical service, anywhere on the network, to keep up with the charge.
* Peace of mind — no data is lost, ever. No more stress. No more panic when something goes wrong in production.
* Lower latency — we got rid of polling processes, cron jobs, batches…
* Streamlined monitoring and debugging.
* Trouble free maintenance — thanks to decoupling we can add, stop, restart or retire services as we want, without disturbing the system.
* Painless release of new features – just add a new service, it's completely transparent to others.
