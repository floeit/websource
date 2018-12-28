---
title: "Terminology"
date: 2018-05-14T17:05:08+01:00
anchor: "terminology"
weight: 90
---

Flows are coordinated by nodes issuing events to which other nodes `listen`.

`Host`    - Any named running `floe` service, it could be many such processes on single compute unit (vm, container), or one each.

`Flow`    - A description of a set of `nodes` linked by events. A specific instance of an flow is a `Run`

`Node`    - A config item that can respond to and issue events. A Node is part of a flow. Certain Nodes can Execute actions.

The types of Node are. 

* `Triggers` - These start a run for their flow, and are http or polling nodes that respond to http requests or changes in a polled entity.
* `Tasks` - Nodes that on responding to an event execute some action, and then emit an event.
* `Merges` - A node that waits for `all` or `one` of a number of events before issuing its event.

`Run`      - A specific invocation of a flow, can be in one of three states Pending, Active, Archive.

`Workspace`- A place on disc for this run where most of the run actions should take place. Since flow can execute arbitrary scripts, there is no guarantee that mutations to storage are constrained to this workspace. It is up to the script author to isolate (e.g. using containers, or great care!) 

`RunRef`   - An 'adopted' RunRef is a globally unique compound reference that resolves to a specific Run.

`Hub`      - Is the central routing object. It instantiates Runs and executes actions on Nodes in the Run based on its config from any events it observes on its queue.

`Event`    - Events are issued after a node has completed its duties. Other nodes are configured to listen for events. Certain other events are emitted that announce other state changes. Events are propagated to any clients connected via web sockets.

`Queue`    - The hub event queue is the central chanel for all events.

`RunStore` - the hub references a run store that can persist the following lists - representing the three states of a run..

* `Pending` - Runs waiting to be executed, a Run on this list is called a Pend.
* `Active` - Runs that are executing, and will be matched to events with matching adopted RunRefs. 
* `Archive` - Runs that are have finished executing.