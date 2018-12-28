---
title: "Quick Start"
date: 2018-05-14T17:05:08+01:00
anchor: "quick-start"
weight: 70
---

[Download](https://github.com/floeit/floe/releases) or build from scratch a `floe` executable, and add a local [`config.yml`](http://www.floe.it/#example)

Start a host processes:

`floe -tags=linux,go,couch -admin=123456 -host_name=h1 -pub_bind=127.0.0.1:8080`

These commands default to reading in the local `config.yml`

web 
---

Open a local browser to http://localhost:8080/app/dash
