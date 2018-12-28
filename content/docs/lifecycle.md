---
title: "Life Cycle"
date: 2018-05-14T17:05:08+01:00
anchor: "lifecycle"
weight: 110
---

When a trigger event arrives on the queue that matches a flow, the event reference will be considered 'un-adopted' this means it has not got a full run reference. A pending run is created with a globally unique compound reference (now adopted) - this reference (and some other meta data) is added to the pending list of the host that adopted it as a 'Pend' - this may not be the host that executes the run later. (These were called TODO's but that has a very particular meaning!)

A background process tries to assign Pend's to any host where the `HostTags` match, and where there are no Runs already matching the `ResourceTags` asked for - this allows certain nodes to be assigned to certain Runs, and to serialise Runs that need exclusive access to any third party, or other shared resources.

Once a Pend has been dispatched for execution it is moved out of the adopting Pending list and into the Active List on the executing host.

When one of the end conditions for a Run is met the Run is moved out of the Active list and into the Archive list on the host that executed the Run.

All of this is dealt with in the `Hub` the files are divided into three:

* `hub_setup.go` - The Hub definition and initial setup code.
* `hub_pend.go` - Code that handle events that trigger a pending run, and dispatches them to available hosts.
* `hub_exec.go` - Code that accepts a pending run and activates it, directs events to task nodes, and Executes tasks.