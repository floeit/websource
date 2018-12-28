---
title: "Introduction"
date: 2018-05-14T17:05:08+01:00
anchor: "intro"
weight: 10
---

Floe is a continuous delivery cluster written in golang. It can be used for all general build and deploy types, not just go. Essentially a golang alternative to jenkins pipelines, and GoCD.

![Flow in progress](/img/floe.png)

All configuration is via yaml files that are expected to come from a repo. There is no config via a web interface - which is hard to version control.


{{% block note %}}
A floe cluster is masterless, so scales well and is robust. Master slave topologies can also easily be created.
{{% /block %}}
