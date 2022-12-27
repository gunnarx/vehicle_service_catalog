VSC Terminology

The purpose of this document is to provoke discussion and eventual rough consensus around the meaning of terms used to describe VSC and its technologies.

Many of these are general Computer Science terms that have been defined in different ways in different places.  Therefore, these definitions will surely not match every system (or personal opinion) out there.  

We can still discuss in order to align these terms as far as is possible with the common-understanding.  When arguing for a different meaning, please provide references to the documented meaning in other related systems.  That should be the basis for alignment.

The document also states a number of quite specific definitions/opinions that go a little bit beyond pure definition of terms. The purpose is to eventually achieve a strict and common understanding of the behavior/semantics of the VSC language and system so that it can be specified clearly.  An example of such a statement would be "There can be only one service per file".  Once again, the purpose is to provoke discussion, for a while, until final decisions are taken.


## Interface Description
- This means all content that is described using **only** the Interface Description Language, not supplementary files.
- A VSC Interface Description is designed to be as independent of the target technology as possible, and as reusable as posible.
- The design principles therefore state that the Interface Description excludes information that is target-dependent or unique for a particular environment.


## Target Environment

- For each type of output result from a generator or conversion tool, there will be environment-specific details to consider, and in this proposed terminology we refer to all of those as simply the "Target Environment".
- Target Environment is a catch-all term to indicate the context and environment of artifacts that are generated from IDL + supplementary information.
- The term means any and all things that are unique about that environment, or looking at it another way - any and all details that cannot universally be determined by reading only the Interface Description only would be part of the Target Environment.  The target environment details are involved in a particular interpretation of the Interface Description.

Example:
1. The chosen programming language in the case of code-generation tools is part of target environment.
2. Also any details relating to specific software technologies that are being targeted.  For example if HTTP is an applicable protocol to use, then the choice of this protocol is part of target environment.  Furthermore, details of transport layer security (TLS) would apply if HTTP is used, whereas for other protocols it might not.  Anything related to TLS is thus part of the HTTP focused target environment - it is not part of the Interface Description.

N.B. In some texts, "target-environment dependent", may sometimes be shortened to simply target-dependent or similar.

DISCUSSION: In the embedded world, "target" often is understood as the embedded hardware.  Is there another better term we can use for Target Environment?


# Namespace

- A Namespace is a way to collect entities into named groups.
- Namespaces are used to ensure that local names will not clash with identically named items in other namespace.
- Namespaces usually also separate what objects can be *seen* or *reached* from within a part of a program.  In particular in VSC, it shall serve that purpose.
- Namespaces can contain other Namespaces, creating a hierarchy.
- How visibility/reachability is handled *might* be specific to the target language or environment, but VSC expects the following behavior to be the normal one unless there are very good reasons for exceptions:
  - Items within different (non-parent) namespaces are logically isolated from each other unless otherwise specified, and these frequently used hierarchy rules apply:  Everything within a child namespace can see and reach items in any of its parent namespaces.  For non-parent relationship, it is required to either specify an item using a fully-qualified "path" through the namespace hierarchy, or to make a statement of reference or inclusion of another namespaces into the current namespace.

## Interface
- Generally, Interface is a broad term.  We are here concerned with functional software interfaces.
- The VSC interface definition language strives to usable in all areas of a computing system.   The VSC Interface definition does not define details around communication hardware, implementation language, data-communication protocols, bit-encoding and in-memory layouts etc.  Such information is however added through supplementary information in the mappings and tools that consume the interface description.  In this environment the content of an Interface are things like 
- A VSC **Interface** is *primarily* defined by a collection of **Methods**, but may also expose functionality through events and data properties/attributes.  The namespace may also include variables, constants, method-arguments, types, observable data-items and events.
- An Interface name is not itself a Namespace but its contents must be defined within a Namespace.
- All Methods defined or referred to by an Interface are required to be within the same Namespace, i.e. it is not possible to build an Interface by referring to Methods that have been defined in different Namespaces.

## Service
- (General) A service is a software functionality or set of functionalities that can be activated by a client. A key characteristic of a service in support of SOA is that it fulfils a fairly specific and limited purpose, as long as it also has low coupling to other services and strives to provide a useful function also if used independently from other services.
- (VSC) A VSC **Service** contains one or several **Interface**s, which are defined or referenced within the service definition.  The service definition may within its namespace also define **Type**s, constants and other definitions required to make use of the service.
- A Service name is not in itself a Namespace, but its contents must be defined within *at least one* Namespace.  The used Namespace is referenced or defined from within the Service definition.
- (VSC) A Service must have its fundamental definition in **one** single .yaml formatted file, although this file might use references to other parts.  In other words, all Interfaces that make up a service must be defined, or "included" from within this file.
- (VSC) A single file contains a maximum of one Service.  NOTE: In certain tool environments, it is of course possible to combine several files into one file before processing the input but that would be a tool-specific handling.

TODO: (To be further determined - how complete is the Include/Reference concept now?)_

## Method
- A method is a single software operation (a.k.a. function) with a (mandatory) name, plus a set of (optional) **input parameters**, **output parameters**, and **error conditions**.
- Like most other objects, a Method is always defined in the context of a Namespace, but this is often implicitly determined because the Method is defined within the context of an Interface/Service

## Errors
- A VSC Error definition is an error condition that can appear while (attempting to) execute a given Method.
- An error is defined by referencing one or more **Type**s that define the data that shall be communicated when an error occurs.  Enumeration types are common, but it could be any defined type.
- There can be more than one error type defined for a method.
- It is not specified by VSC how errors are propagated to the caller of a function - that translation is target environment specific.  In other words it might be a return or output parameter in one programming language but using **Exception**-handling in another.  Communication protocols might embed errors in a response message, or implement some other side-channel for error propagation, or other methods.

TBD: Should error also be defined by its name (unique, at least within the namespace), or is it enough to reference its type.  Is this error name reusable across methods, or is it enough that the datatype(s) are reused by methods?


## Property

- A **Property** is an observable data item.  It belongs in a **Namespace**, has a canonical name in addition to possible **aliases** (i.e. alternative names), and a data type.
- A Property is defined as a single item but the data type could be an arbitrary data type, including composite data types.
- The singular nature of a property suggests that it is generally expected behavior that its value shall updated and communicated as an atomic operation, i.e. it is either fully updated or not at all.
- The available features for update and communication are target-environment specific, but the most common expectation is that a Property can be updated and _observed_, meaning that a client can call a method to get or set the current value of the property (with appropriate access control rules) and often also _subscribe_ to be notified of when the value of a property was changed for any reason.
- In some other language definitions this type of item is called an **Attribute** or **Field**.
- A property can also be seen as analogous (and bi-directionally translatable, if datatypes allow it) to a sensor Signal in the Vehicle Signal Specification (VSS).

**Side proposal**: Consider renaming Property to Attribute to match Franca IDL, and because properties have another use in OpenAPI and AsyncAPI, and it seems other HTTP-related communication as well.

## Return values

- A Return Value specifies what is returned from a Method execution.  Although "out parameters" could serve a similar purpose, the Return Value has been given special status.
- Only one return value type is defined per method.
- NOTE: Please be aware that while return-value is often traditionally used for indicating success/error, the generalized Errors concept is more powerful and should often be preferred.
- There is only a single return value type, but it can use any data type, including composite types.
- The term Status may be used in other technologies and the meaning of return value shall be seen as equivalent of status.

### Return/status communication in asynchronous vs. synchronous method calls

- In computing systems there are two main paradigms regarding invocation of a procedure/function: Blocking/synchronous vs. Non-blocking/asynchronous.

  - In a blocking/synchronous environment, the client that invokes an operation (i.e. calls a function) will halt until the operation is completed and "returns".  Upon return, a return-value (result) may be communicated back to the client, and it then continues its normal execution flow.

  - In an asynchronous setup, a client can trigger the start of an operation and then continue doing other work while the method executes in parallel.  In this case, the executing function will report back when the operation is finished, but it could also give continuous reports as to the progress of the function ("streaming updates").

- In the VSC-language, methods can be defined without defining their invocation strategy (synchronous/asynchronous). It is more flexible to define this separately in a deployment-model/target-mapping because it leads to interface descriptions that can be reused in a wider set of circumstances.  The desire for sync vs. async may change in different circumstances, and the availability of sync/async might be constrained by the target environment capability.

- In all cases, a Method in VSC-IDL-language (optionally) defines a Return Value type.  VSC-IDL-language has the unique feature of using the Return Value type to serve different purposes in sync vs async situations:

- In this manner, the same concept (return value) can be defined for the interface, without knowing the manner that the function will be invoked (e.g. the chosen communication protocol)

- Although some details are often target-dependent, the following defines the expected behavior of the return typein a system built from a VSC interface:
  1. In the case of a blocking/synchronous call, the Return Value type is the value communicated once after the operation completes.
  2. In the case of a non-blocking/asynchronous call, the same defined Return Value type is also used with the difference that it *may* be returned _multiple times_.  Using this behavior is optional and target-dependent, but if it is used then a target environment may choose to provide a regular stream of status updates while the method executes.  In each such update, a value is transferred that matches the defined Return Value type**.

  - It follows that only valid values of the Return Value type shall be returned. 

  - \*\* Design note:  This is already in itself a very powerful mechanism and should suffice in a majority of cases, but if any additional status/streaming-data functionality is required, that may of course be done by defining a data Property in the interface, whose value is some kind of status update for the executing method.  There is no *formal* way to describe the relationship between Property and Method in this secondary approach - it would rely on documenting the relationship.

  - \*\*\*Design note 2: While the interface can now be defined without deciding up front about synchronous/asynchronous operation, if an async operation _may_ be expected it is still a good idea to design the return value for this.  It would probably include something to indicate that processing is under way.  For example the streaming returns during an operation might be: Started, Processing..., Processing..., Processing..., Done!"`  (Note that adding/changing return value can, like most features, be extended by overlays.)

- After an operation has been completed and its equivalent of "DONE" has been communicated, the function is expected to seize any further communication of the return/status type.

- A return value can naturally indicate that the operation did NOT complete successfully, but the more capable Errors concept should not be overlooked.


## Event

- An Event is a time-sensitive communication with _fire-and-forget_ behavior, sent between parts in the distributed system.
- A VSC Event can carry multiple pieces of data with it, each of which can use an arbitrary data type.
- A VSC Event definition actually looks similar to a Method but with some differences in definition and behavior:
  - An Event has a name, and optionally a set of parameters that are distributed with the event, each having a name and a datatype.  This is something that mimics a Method signature.
  - An Event can however **not** have a Return type, nor any out-parameters.
- _Fire-and-forget_ expected behavior means that on the VSC semantics level it is not expected to be a guaranteed delivery.  It is decided by the target environment mapping whether an Event can be guaranteed to be received or if it is non-reliable.  In either case, there is **no reply** defined for an Event.  A Method call shall be used where reliability is required, in which case a return value would indicate that the request was received and completed.
- A VSC Event is a separate and expressly defined object in the interface description.  It is therefore not (in our particular definition) to be mixed up with any other "happenings" in a system that are not defined inside a VSC/interface definition.  It shall not be confused with happenings/messages that are _automatically_ sent by some target environment because of the defined behavior of its objects.  For example, in some systems, subscribing to changes of an observable data Property, may trigger something that some might call "update events", but that is a consequence of the behavior of the observable Property, and is not to be confused by expressly defined Event objects. It is simply not needed to define any VSC Events for such messages that are created automatically by the particular data protocol or target environment mapping.

- A VSC Event is also not a persistent data item, and thus has no concept of active-low, edge-trigger, rising/falling edge, etc. (other than what its name and documentation indicates\*\*).  The Event is simply transferred by the sender when the sender thinks it is appropriate, and the documentation would explain what the meaning is of this particular event.

- \*\* Design tip:  If there is a desire to model On/Off events, and avoid the verbosity of duplicating everything into two different Event objects -> simply give the event a single name and transfer a boolean as argument: FooState(bool) where true/false indicates the on/off state.  Another option, as always, is to define an observable data Property and use the target environment's Pub/Sub capability to get updates.  (Depending on the guarantees given by the target environment, the latter might give reliability guarantees that Events do not have).

- An Event can be sent from client to server or from server to client(\*).
(\*) TODO: Shall the direction of an event be defined somehow?

- \*\* If an event is to be transferred as a "broadcast" in a particular environment whereas other Events would be addressed uniquely to a particular client, then define this difference as extra information in a deployment model / target mapping.  It is not specified in the interface description.


