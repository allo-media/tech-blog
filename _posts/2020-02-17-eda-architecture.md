---
layout: post
title:  "An event driven architecture"
excerpt: "Why and How we ditched our old architecture based on synchronous REST services for a completely asynchronous event driven architecture."
date:   2020-02-07 12:53:00 +0100
categories: architecture
tags: python services rust broker event bus rabbitmq architecture
---

## Introduction

At Allo-Media, like many other businesses, our value chain looks like a pipeline: we collect conversations (mainly through phone) sent to us by our customers, we transcribe them, we tag the transcripts with named entities, we anonymize both the transcript and the audio, then we qualify the content with semantic tags, and finally we index them and provide a UI and API to consult, search and analyze the conversations. All those steps are completed automatically by NLP and AI algorithms.

Such pipelines are well suited for service based architectures. If you need to add a new feature, you introduce a new service into the pipeline.

Our first take at it was based on REST services. Unfortunately, that approach had many drawbacks:
* it introduces strong coupling between components, as each service has to know about the other related services, their addresses, their purposes, their APIs…;
* load balancing requires ad-hoc solutions;
* high availability is tricky because of the synchronous communication: if the requested service is down, the caller has to implement complex "retry later" strategies or give up! And so on for each service.
* upgrading or adding new services is a lot of work as it impacts other services and requires careful coordinated releases. Plus, you have to provide them with IP and DNS addresses.

So one year later, as our activity grew and development accelerated, we quickly realized that we needed:

* maximum service decoupling;
* easy distribution;
* no brainer load balancing;
* one to one, one to many, many to one, many to many asynchronous communications;
* high availability: hot restart of services, transparent addition or removal of service instances (workers), resilience to (reasonable) downtime of some services;
* support for heavy payloads (megabytes of mp3 audio);
* no data loss, whatever happens.

## Enter the event driven architecture

The best way to achieve those goals is to free your mind from the classical pipeline point of view and instead see the value chain as an ecosystem of business services, each focused on providing a specific value and reacting to events (inputs) and producing new events (outputs). This new metaphor has not only technical benefits, but also business and organizational ones. By reasoning in terms of business units of your value chain, its easier to identify the people involved, the business experts who are the references for the job, the exact value added by the service, etc…

Events are precisely defined messages that streams on a message bus, and each logical service (implementing one such business service as explained above) subscribes to the events that are relevant to it, without needing any knowledge about what produced them and how. They also push their own events on the bus, without caring about what consumes them.

In that way, we completely decouple the services between each other and the message broker running the message bus provides us with load balancing, distribution and high availability for free. For that purpose, we chose [RabbitMQ](https://www.rabbitmq.com/) as message broker for its reliability record and ease of use.

Now, the messages *are* the API, the only business and technical reference.

After much thinking and experiments, we came with these core design principles that are very important, and after 4 months of production use, we are very glad we complied with them from the start:

### Business Services and Data Processing Services

We make a distinction between Business Services and Data Processing Services (a.k.a. utility services) to cleanly separate business logic from data processing complexity.

DP Services are expected to be  **pure, stateless, services** that provide some kind of algorithmic data processing (computations, transformations, …). Moreover, they are also context free: they should not depend on business rules, assumptions or external data sources. All they need to do their processing must be in the command itself. They should not have to query a tier to get more data. DP services are kind of *universal* libraries and can even be provided by tiers.

Business Services implement the customers' workflows and only focus on business rules and requirements to orchestrate and implement the value addition upon our customers' audio and data. They make use of the DP services as a library for that. There are very specific to us: you would never want to externalize your business services.

Business Services persist the data they produce and are their unique trusted source of truth.

Business Services build and maintain their own customer configuration from events on the bus.

All services must be idempotent, that is, if they receive twice the exactly same message, they must behave identically and produce the same outputs.

### Commands, Results and Events

In the same way we have two different kinds of services, we have two different kinds of messages: *Commands* and *Events*.

Events are business messages emitted by business services and telling the bus what happened.

A business service owns the type of Events it emits. It knows nothing about the services who will process them. It subscribes to the types of Events it needs but knows nothing about their origins.

An Event type defines the meaning of the events of that type and their data schema. They must be documented.

The type of an actual event message is given by its name (used as *routing key* in [RabbitMQ AMQP protocol](https://www.rabbitmq.com/protocol.html), see *Topology* below). The event type name must be in the form `SubjectPastParticiple`. For example, `ConversationStarted`, `CustomerCreated`, `ShoppingCartValidated`… If you're not able to immediately give a name to your event type, it means it is not well defined, or that it is not an event. Maybe, you need to refine your service or split it, as you may not have analyzed your value chain deeply enough?

Commands are utility messages consumed by Data Processing Services.

A DP Service owns the types of the commands it consumes and their results. A command is always addressed to the logical service that owns it.

A Command type defines the meaning of the command, its data schema and its result data schema. It must be documented.

The type of an actual command message is given by its name. It is in the form `VerbObject`. For example: `AnnotateText`, `TranscribeAudio`…
As commands are addressed to a particular logical service, the *routing key* of a command is in the form `logical_service_name.commandname`. For example: `voxo.TranscribeAudio`.

The command contains the return "address"  (`reply_to`) to which the result is to be sent and a reference set by the sender that is returned as-is (the `correlation_id`). That reference is very important for the sender: as all communications are asynchronous, the service requesting the command needs a way to reconcile the received result with the initial request it made.

The `correlation_id` and `reply_to` are standard on Commands and Results, and thus, transported into the AMQP headers (in which they are already standard) instead of the message payload which is service specific.

A Result contains the result of the process, that can be the successful outcome, or an error.

Error results are expected and documented: they are “normal” errors, not bug reports. Bug exceptions *must not return an error result*. In case of unexpected error, the service will replace the input command on the queue once, and if a second try raises an unexpected error again, the message is refused and goes into the dead letter queue for investigation. The exceptions are always logged.

We can also have logging messages, on their own exchange (see below), to easily collect application logs.

All messages contain a `conversation_id` which has the following properties:
- it is unique in time;
- it is created by an *Event* (never by *Commands*) that is published for reasons external to the bus and not as a reaction to other *Event*s; we call that *Event* the *initial event*.
- Any Event, Command, Result created as a reaction to another message *M*, takes and repeats the `conversation_id` of *M* as is.

All consequent messages of a given initial event share the same conversation id, and no other event does. That way, we can easily trace and debug the actual pipeline of each incoming conversation for example.

Messages must be forward compatible:

- a new version of a Message schema for an application can add fields but must not remove or redefine existing ones;
- the implementation of a Message decoder must ignore unknown fields without crashing.

The detailed documentation of the actual messages must be kept up to date in an easily reachable place by the developers.

### Topology

The topology describes how the bus is implemented on the message broker. In RabbitMQ, this is done by instantiating exchanges and queues.

In Rabbitmq parlance, exchanges are kind of routers, and queues are bound — using subscriptions to particular *routing keys* — to them by client applications to store their messages waiting for processing. It is a good practice to consider the queues as private to the logical service (but are shared by the workers of that logical service). We use the message type name as routing key.

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

The message processing acknowledgment by clients is a very useful mechanism to ensure no data is lost — that is, a message is guaranteed to be processed at least once — and to allow efficient load balancing. Indeed, the message broker won't remove a message from the queue until the client who took it for processing tells it has finished its job. During this time, the message is reserved. If the worker who took the message goes down before acknowledging, the message becomes available for any other worker, or the same worker when it comes back. Moreover, the broker knows at any given time which workers are busy and which ones are idle, so it can better share the load between them.

Note that this is the base topology. As the topology is **created by the services themselves** instead of a central configuration, they can extend it locally (i.e. on their side) for their own needs.
That also means they must all agree on the base topology exposed above to join the bus.

To avoid code duplication, we developed a python mini-framework that implements this base topology and all the desired behaviors of the services. We released it under a MIT license as [Eventail](https://github.com/allo-media/eventail). We are also working on a mini-framework for Rust.

#### Dead letters

We use [RabbitMQ policies](https://www.rabbitmq.com/dlx.html) for that.
Services just have to `nack` the message with the `requeue` flag set to false and the broker takes care of the rest. The framework manages this automatically in case of an unexpected exception raised by the user code.

There should be something listening to the dead letters though.

### Monitoring

Beside the "classical" network and server monitoring solutions, we use more high level, specific tools for monitoring our event driven architecture.

For a global bus health monitoring, we use the [RabbitMQ management plugin](https://www.rabbitmq.com/management.html). It provide a live view of what's happening on the bus:
* message throughput;
* queue states;
* broker resource usage;
* client connection states…


As we said earlier, the services send their application logs on the bus, on a dedicated exchange. The [Eventail](https://github.com/allo-media/eventail) framework formats the log messages in the GELF format used by [Graylog](https://www.graylog.org/) so we can leverage that log management solution which can natively connect to AMQP brokers like RabbitMQ. We can then use Graylog to define alerts, create graphical dashboard on the business processes, search the logs for debugging (by `conversation_id` for example), etc…

In addition to application logs, the services also send periodical health checks on the same exchange, and those are also collected in Graylog: at any moment we know which services or workers are down, or are running in a degraded state.

### Testing, debugging and "repair"

The Eventail framework provides command line utilities for testing, debugging and replaying messages in the dead letter queue.

The testing commands allow the developer to publish events on the bus, or to send a command to a service and display the result on the console.

The debugging commands allow the developer to target and display logs on the console, or to subscribe to events and commands in order to display their content as they occur. There is also a command to inspect the content of a queue (without consuming its messages).

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
