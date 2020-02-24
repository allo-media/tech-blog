---
layout: post
title:  "An event driven architecture — part 5"
excerpt: "Outcome."
date:   2020-04-25 12:53:00 +0100
categories: architecture
tags: services event bus architecture
---

## Outcome

> What do we get?

After four months in production, our event driven architecture brought us:

* Scaling — just start a new instance of a logical service, anywhere on the network, to keep up with the charge.
* Peace of mind — no data is lost, ever. No more stress. No more panic when something goes wrong in production.
* Lower latency — we got rid of polling processes, cron jobs, batches…
* Streamlined monitoring and debugging.
* Trouble free maintenance — thanks to decoupling we can add, stop, restart or retire services as we want, without disturbing the system.
* Painless release of new features – just add a new service, it's completely transparent to others.
