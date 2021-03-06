---
layout: "docs"
page_title: "Commands: Event"
sidebar_current: "docs-commands-event"
description: |-
  The event command provides a mechanism to fire a custom user event to an entire datacenter. These events are opaque to Consul, but they can be used to build scripting infrastructure to do automated deploys, restart services, or perform any other orchestration action. Events can be handled by using a watch.
---

# Consul Event

Command: `consul event`

The `event` command provides a mechanism to fire a custom user event to an
entire datacenter. These events are opaque to Consul, but they can be used
to build scripting infrastructure to do automated deploys, restart services,
or perform any other orchestration action. Events can be handled by
[using a watch](/docs/agent/watches.html).

Under the hood, events are propagated using the [gossip protocol](/docs/internals/gossip.html).
While the details are not important for using events, an understanding of
the semantics is useful. The gossip layer will make a best-effort to deliver
the event, but there is **no guarantee** delivery. Unlike most Consul data, which is
replicated using [consensus](/docs/internals/consensus.html), event data
is purely peer-to-peer over gossip. This means it is not persisted and does
not have a total ordering. In practice, this means you cannot rely on the
order of message delivery. An advantage however is that events can still
be used even in the absence of server nodes or during an outage.

The underlying gossip also sets limits on the size of a user event
message. It is hard to give an exact number, as it depends on various
parameters of the event, but the payload should be kept very small
(< 100 bytes). Specifying too large of an event will return an error.

## Usage

Usage: `consul event [options] [payload]`

The only required option is `-name` which specifies the event name. An optional
payload can be provided as the final argument.

The list of available flags are:

* `-http-addr` - Address to the HTTP server of the agent you want to contact
  to send this command. If this isn't specified, the command will contact
  "127.0.0.1:8500" which is the default HTTP address of a Consul agent.

* `-datacenter` - Datacenter to query. Defaults to that of agent.

* `-name` - The name of the event.

* `-node` - Regular expression to filter nodes which should evaluate the event.

* `-service` - Regular expression to filter to only nodes with matching services.

* `-tag` - Regular expression to filter to only nodes with a service that has
  a matching tag. This must be used with `-service`. As an example, you may
  do "-service mysql -tag slave".

* `-token` - The ACL token to use when firing the event. This token must have
  write-level privileges for the event specified. Defaults to that of the agent.
