# Reactive Hypermedia: Design Patterns for Asynchronous and Reactive Machine-to-Machine Interaction using Hypermedia Controls

## Introduction

In the last two articles, I discussed some design issues and patterns in hypermedia controls for machine interfaces, and presented an experimantal open source software platform and demonstrator for an example hypermedia system.

In this article, I would like to summarize the progress on this topic over the last year and focus on the subject of asynchronous and realtime interactions using REST design principles and, in particular, hypermedia controls.

To set the background for this discussion, I would like to take a look at REST system design in the context of state machine design.

In REST system design, a distributed system is represented as a state transition graph. The nodes of the graphs represent states of the system, and the arcs of the graph represent possible state transitions. [example]

## CRUD, REST, and Hypermedia

The common resource design pattern known as CRUD exposes the system state contained in the nodes of the graph. Resource state may be Created, Retrieved, Updated, or Deleted.

Hypermedia controls expose the system state transition model at the edges of the graph. Hypermedia controls are the hyperlinks and submission forms, or their machine equivalents, that describe the available state transitions offered by the system, and how, from the client, they may be effected.

To bring in another related set of concepts, hypermedia controls implement a set of "control plane" abstractions that work together with the "data plane" abstraction of CRUD, forming two aspects of a distributed system architecture.

Without hypermedia controls, CRUD clients and servers will generally need to be more tightly coupled. These systems tend to use early binding of clients to servers, with schemas and introspection, code generation, and related techniques to drive client state machines to consume resources.

With hypermedia controls to dynamically describe state transitions, REST clients can consume the hypermedia controls in order to understand how to drive system state transitions, in a more loosely coupled, separately evolvable, late binding way.

Hypermedia controls describe possible state transitions based on the current state of the system, rather than pre-determined by a static resource design.

## Reactive Hypermedia: Actions and Link Bindings

In the last article, a class of hypermedia controls was described that facilitates asynchronous and realtime interaction between hypermedia clients and REST servers. These belong to a general class of Reactive Hypermedia controls, that can direct the transfer and processing of dynamically changing data and asynchronous events.

This article describes two types of this class of Reactive Hypermedia controls, Actions and Link Bindings, in more depth, with examples and terminology consistent with the HSML Internet Draft published in the IRTF Thing to Thing research group (T2TRG):

## HSML: Machine Hypermedia Content Formats

Since the last article, there is a new Internet Draft describing a content format for hypermedia collections on constrained networks. HSML is a simple representation and interaction model for collections and associated links, forms, and items, using CoRE Link-Format and SenML.

https://datatracker.ietf.org/doc/draft-koster-t2trg-hsml/

### Common Transfer Layer

The HSML draft discusses a common abstract transfer layer using CRUD + Observe/Notify, also known as CRUD+N, which is used to map various concrete protocols, including HTTP, CoAP, and MQTT. 

This supports a common model for resource state interaction, upon which a consistent hypermedia interaction model may be constructed, and from there be extended.

## Asynchronous Interaction 

### Problem statement and scope

Asynchronous interaction over REST involves use cases for asynchronous state transfer in both directions; from client to server, and from server to client.

In the client to server case, the problem is in describing incoming state transitions which may or may not have immediate results. This enables the client to direct reactive server processing of actions.

In the server to client case, the problem includes describing how to use asynchronous communication to communicate resource state changes outgoing, in real time, to clients in order to enable reactive client processing of server state changes and events.

### Actions and Forms

Incoming state transitions are state transitions that a client wishes to make on a resource, such as turning a power switch on or opening a garage door.

For incoming state transitions, the problem is how to inform the client what transitions are available and how to construct requests which change the resource state of the system in some way that may be more or less indirectly related to the control input. It is not always a simple and immediate CRUD state update. The state of the resource may need to be asynchronously monitored to complete the client state transition at some time in the future.

For example, an actuator may take some time to complete its action, like closing a motorized gate or garage door. The outcome may be uncertain, as in the case of power interruptions or mechanical blockage of the gate. These conditions result in a number of possible eventual outcomes which need to be returned to the client for processing and client state update.

A meta-model using the concept of an "action" is presented in the HSML draft. An Action describes and accepts a representation for a desired state change to a resource. The conceptual model is that the desired state change is "created" in the context of an Action resource, which then affects the state of the linked resource(s), and which can then be monitored for its eventual outcome.

An Action link works in analogous fashion to a form in HTML; it dscribes what the action is expected to accomplish, like "turn on the light" or "add a post to the blog", and includes information on how to accomplish the action, like what method to use, which resource type, which content format, schema information to describe the payload, and descriptions of expected responses.

Action links can describe simple action operations that use REST state updates, or more complex actions which create representations of abstract action descriptions in collections of action instances. Such collections may be used to sequence submitted actions in FIFO or priority order, and for clients to track long running or pending actions which have not yet completed.

Actions may be performed directly on resources they are intended to update the state of, or they may be performed on proxy resources, as in the likely example of action instances crreated in a collection.

In the case that Actions are performed on a proxy resource, the resource may be indicated in the context of the affected resource(s) by including a link with the relation type "alternate" and with a resource type indicating "action".


### Link Bindings

For outgoing state transitions, there is the additional problem of describing how a client is expected to asynchronously obtain the state transitions as they occur, perhaps with some additional QoS parameters specified as system constraints.

The concept of a Link Binding is presented in the HSML draft. A link binding, in this context and in the context of the CoRE dynlink Internet Draft, is a hyperlnk that defines a dynamic data update from one resource to another.

The link source and destination are generalized to allow resource state to be communicated using an extensible set of transfer protocols. For example, links using the "mqtt:" scheme will instruct the system to publish updates  from the linked resource to an mqtt broker, or subscribe to an mqtt broker to obtain state updates for the linked resource.

The description of link bindings assumes that there is a generalized REST-hook mechanism in place for the resource state to be monitored and used to trigger the link binding transfer operation. CoAP Observe may be used if available, or HTTP EventSource, or MQTT Subscribe.

The link binding can be located with the information source and cause information to be pushed or published to the destination resource, or it may be located with the destination resource and cause the source resource to be subscribed, observed, or polled. 

The link binding may be in fact stored and operated in a third place, separately from the source or destination, and mat use an agent at the third party to effect the source and destination transfer operations. 

Link bindings may be defined with separate source and destination controls for transfer layer perameters like methods, content formats, and resource type queries. This enables link bindings to be generally used for resource statre transfer and for triggering Actions or generating abstract events.

Link bindings may have different source and destination schemes, enabling link bindings to be used to convert protocols. For example, updates to a REST resource may be published to an MQTT broker using a link binding. Since the content format and other resource information can be obtained using HTTP, there is less need to augment the MQTT system with meta-data when used in this way.

For schemes that don't offer subscribe or observe, for example HTTP, RESThooks can be created and used in a structured way. Similarly, for systems using MQTT subscribe or CoAP Observe, link bindings enable the orchestration and inspection of dynamic resource interactions using graph techniques and tools.

An extension to link bindings may be created which enables a link binding to consume a hypermedia action. This would enable cross-protocol adaptation without proxies, by using dynamic adaptation code in the libraries that consume the hypermedia controls.

## System architecture

Reactive hypermedia controls enable the dynamic orchestration of client-server interactions by discovering actions and adding dynamic link bindings to connect resources, event sources, and event handlers. The following are some examples of system-level orchestration using reactive hypermedia.

### Actions in hypermedia clients

### Link bindings to monitor server state using REST callback 

### Using pubsub communication with REST

### Device-to-device orchestration using link Bindings


## References

### Roy Fielding's Reference Work on REST
https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm

### Roy Fielding's note on Hypertext and REST APIs
http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven

### T2TRG 
https://datatracker.ietf.org/rg/t2trg/charter/

### HSML
https://datatracker.ietf.org/doc/draft-koster-t2trg-hsml/

### RFC 6690
https://tools.ietf.org/html/rfc6690

### CoRE SenML
https://tools.ietf.org/html/draft-ietf-core-senml-05

### CoRE Dynlink
https://tools.ietf.org/html/draft-ietf-core-dynlink-03

### Even Models for RESTful APIs
http://iot-datamodels.blogspot.com/2013/05/event-models-for-restful-apis.html

### REST to MQTT Bridge slides
https://www.slideshare.net/michaeljohnkoster/mqtt-rest-bridge