---
layout: post
title:  "An event driven architecture — part 2"
excerpt: "Core design principles."
date:   2020-03-04 12:53:00 +0100
categories: architecture
tags:  services event bus architecture
---

In the previous post in this series, we explained why we ditched our old architecture based on synchronous REST services for a completely asynchronous event driven architecture.

Today, we address the core design principles that were crucial in the success of this enterprise.

## Business Services and Data Processing Services

We make a distinction between *Business Services* and *Data Processing Services* (a.k.a. utility services) to cleanly separate business logic from data processing complexity.

DP Services are expected to be  **pure, stateless, services** that provide some kind of algorithmic data processing (computations, transformations…). Moreover, they are also context free: they should not depend on business rules, assumptions or external data sources. All they need to do their processing must be in the message they receive. They should not have to query a tier to get more data. DP services are kind of *universal* libraries and can even be provided by tiers.

Business Services implement the customers' workflows and only focus on business rules and requirements to orchestrate and implement the value addition upon our customers' audio and data. They make use of the DP services as a library for that. There are very specific to us: you would never want to externalize your business services.

Business Services persist the data they produce and are their unique trusted source of truth.

Business Services build and maintain their own customer configuration from events on the bus.

All services must be idempotent, that is, if they receive twice the exactly same message, they must behave identically and produce the same outputs.

## Commands, Results and Events

In the same way we have two different kinds of services, we have two different kinds of messages: *Commands* and *Events*.

Events are business messages published on the bus by business services and telling the world what happened.

A business service owns the type of Events it emits. It knows nothing about the services who will process them. It subscribes to the types of Events it needs but knows nothing about their origins.

An Event type defines the meaning of the events of that type and their data schema. They must be documented.

The type of an actual event message is given by its name (a.k.a. *routing key* because it is used by the subscription routing). The event type name must be in the form `SubjectPastParticiple`. For example, `ConversationStarted`, `CustomerCreated`, `ShoppingCartValidated`… If you're not able to immediately give a name to your event type, it means it is not well defined, or that it is not an event. Maybe, you need to refine your service or split it, as you may not have analyzed your value chain deeply enough?

Commands are utility messages consumed by Data Processing Services. Imagine an order you pass to a provider. You don't know who exactly will completed it, you don't know how and when either, but you'll get what you want in you letter box sometime later.

A DP Service owns the types of the commands it consumes and their results. A command is always addressed to the logical service that owns it.

A Command type defines the meaning of the command, its data schema and its result data schema. It must be documented.

The type of an actual command message is given by its name. It is in the form `VerbObject`. For example: `AnnotateText`, `TranscribeAudio`…
As commands are addressed to a particular logical service, the *routing key* of a command is in the form `logical_service_name.commandname`. For example: `voxo.TranscribeAudio`.

The command contains the return "address" to which the result is to be sent and a reference set by the sender that is returned as-is, along with the command outcome. That reference is called the *correlation identifier* and it is very important for the sender: as all communications are asynchronous, the service requesting the command needs a way to reconcile the received result with the initial request it made.

A Result contains the result of the process, that can be the successful outcome, or an error, and the correlation identifier.

Error results are expected and documented: they are “normal” errors, not bug reports. Bug exceptions *must not return an error result*. In case of unexpected error, the service will requeue the input command to retry it once, and if a second try raises an unexpected error again, the message is refused and goes into the dead letter queue for investigation. The exceptions are always logged.

We can also have logging messages, on their own exchange (see below), to easily collect application logs.

All the messages that “cascade” from the same source event, share a common identifier, called the *conversation identifier*, which has the following properties:
- it is unique in time;
- it is created by an *Event* (never by *Commands*) that is published for reasons external to the bus and not as a reaction to other *Event*s; we call that *Event* the *initial event*.
- Any message (Event, Command or Result) created as a reaction to another message *M*, takes and repeats the conversation ID of *M* as is.

All consequent messages of a given initial event share the same conversation id, and no other event does. That way, we can easily trace and debug the actual pipeline of each incoming conversation for example.

Finally, the message schemas must be forward compatible:

- a new version of a Message schema for an application can add fields but must not remove or redefine existing ones;
- the implementation of a Message decoder must ignore unknown fields without crashing.

The detailed documentation of the actual messages must be kept up to date in an easily reachable place by the developers.


In the next post in this series, we'll see how we implemented those principles and behaviors in the actual architecture.

