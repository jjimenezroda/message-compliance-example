# Message naming guidelines

---
**Author**: `Javier Jiménez Roda [Lead Software Architect]`

**Modification history:**

- **v1.2** by eglif: `Added Message versioning`
- **v1.1**: `Added Property naming guidelines`
- **v1**: `Initial version`

---
**Table of Contents:**

- [Message naming guidelines](#message-naming-guidelines)
  - [Introduction \& Scope](#introduction--scope)
    - [Out of Scope](#out-of-scope)
  - [Problem Statement](#problem-statement)
  - [Background](#background)
    - [Domain Events](#domain-events)
      - [Some examples of event naming following the previous guidelines](#some-examples-of-event-naming-following-the-previous-guidelines)
      - [Examples for long-running operations](#examples-for-long-running-operations)
  - [Guidelines](#guidelines)
    - [General Message naming guidelines](#general-message-naming-guidelines)
      - [Commands](#commands)
        - [Command Acknowledgement](#command-acknowledgement)
      - [Events](#events)
        - [Event representing 'simple' facts](#event-representing-simple-facts)
        - [Event representing the starting/failure or progress of long-running operations](#event-representing-the-startingfailure-or-progress-of-long-running-operations)
        - [Event representing the completion of long-running operations](#event-representing-the-completion-of-long-running-operations)
      - [Requests / Responses (Queries)](#requests--responses-queries)
      - [Property naming guidelines](#property-naming-guidelines)
        - [Boolean properties](#boolean-properties)
    - [Message versioning](#message-versioning)
  - [FAQ](#faq)
    - [Why do we need to follow these guidelines?](#why-do-we-need-to-follow-these-guidelines)
    - [Why not using a fixed naming-conventions for event and commands instead of a domain-based approach as described in this document?](#why-not-using-a-fixed-naming-conventions-for-event-and-commands-instead-of-a-domain-based-approach-as-described-in-this-document)

---

## Introduction & Scope

This document describes the guidelines to be followed when naming messages in the different MolLab Instrument Software Systems. The guidelines **applies to messages sent over a Message Broker** (i.e. RabbitMQ, Mosquito, Kafka, RedPanda, ZeroMQ):

- events
- commands
- requests/responses (queries)

This document applies to each Software Unit which communicates with other software units (or consumes messages from other software units) via a Message Broker. This includes Software units developed in any programming language (e.g. C#, C++, TypeScript, Python, Java, etc.).

### Out of Scope

This document ***does not cover***:

- naming guidelines for HTTP REST endpoints.
- naming guidelines for WebSocket-based messages (e.g. SignalR, Socket.io, etc.).
- RLX OSAL Messages are also left out of this document because these messages are owned and defined by the RLX team.
- naming guidelines for CSM to SubCSM messages nor firmware level messages.

## Problem Statement

It has been detected, during the development of different instruments, that different teams follow different approaches to name events/commands/requests&responses messages produced/published by different software units. Despite the efforts put on reviewing these messages, it is clear (and it was requested by the teams) that some guidelines are missing. Since message definitions are the public data interfaces of the MolLab Instruments, external consumers of the data do also appreciate having homogeneity in the approach for naming messages.

## Background

!!!Hint For information only purposes
    If you want to directly jump to the guidelines, please go to the [Guidelines](#guidelines) section. This section provides some background information about Domain-Driven Design and Domain Events.

Domain-Driven Design is the modeling/design approach used in MolLab for the Instrument Software. Out of Domain-Driven Design there are a set of principles/recommendations on the naming level.
During the development of cobas 5800, a DDD implementation guideline was developed and refined at the starting of cobas 6800/8800 v2.0. The following segment is extracted from there:

### Domain Events

> When an operation is executed at aggregate level (starting from the AggregateRoot) very likely this operation will end up changing the state of one or more of the entities within the Aggregate. When this occurs, the entity whose state has changed must publish a Domain Event informing the rest of the system about what has changed.

> A Domain event is an event that **is of relevance to the Business of the system**. It is published in the context of a concrete Domain and it might very likely be consumed by different components belonging to different Bounded Contexts.
A Domain Event ***expresses a Domain fact (a fact that is of relevance to the Domain of the system) and it is made available to anyone in the system who might be interested.***

> Once a domain event has been published, it can be received by one or more domain event handlers that in turn may trigger additional processing and new domain events, etc. The publisher is not aware of what happens with the event, nor should the listener be able to affect the publisher (in other words, publishing domain events should be side-effect free from the point of view of the publisher). Because of this, it is recommended that domain event handlers do not run inside the same transaction that published the event.

**Domain Event messages MUST:**
> - **Reflect the change of state of an entity** or process manager in its name.
> - Its name **reflects a fact** that occurred in the system.
>   - They reflect Business or system relevant facts.
>   - Avoid using events to communicate technical facts like:
>       - Thread started or finished
>       - Completion of low-level functions (i.e. *AcquireLockCompleted is not a Business domain event if it means a OS-level lock*)
> - The name of an event **MUST** be meaningful
>   - What does this mean?
>       - A human must be able to read the name of an event and understand what it expresses.
> - Their names are **formulated in past-tense**
>   - Examples of good event names:
>       - *RackLoaded*
>       - *RackUnloaded*
>       - *RackHandlerMoved*
>       - *ClotDetectedDuringAspiration*
>   - Examples of **bad** event names:
>       - *EventData*
>       - *InitializationSuccessful*
>       - *IC_ReceiveThreadOnlineFired*
> - It **must specify the identity of the entity whose state has changed**.
>   - It is very important that the consumer of an event can clearly identify the entity whose state has changed.
> - **Domain Events are Immutable**.
> - They contain a Timestamp capturing when the event occurred (not when it was send to the Message Broker. It must represent the exact point in time when the state of the entity was changed or, in other words, when the fact occurred).

> From a design point of view, the biggest advantage of domain events is that they make the system extendable. One can add as many domain event listeners as needed to trigger new business logic without having to change the existing code. This naturally assumes the correct events are published in the first place. Some events one might be aware of upfront, but others will reveal themselves further down the road. One could try to guess what types of events will be needed and add them to the model, but this also risks clogging the system with domain events that are not used anywhere. A better approach is to make it as easy as possible to publish domain events and then add the missing events when one realizes that they are needed.

From Domain-Driven Design point of view there are other recommendations when it comes to event naming:

> In Domain-Driven Design (DDD), naming is crucial for clarity and understanding within the context of the business domain. When it comes to naming events, it's important to follow some guidelines to ensure consistency and effective communication. Here are some guidelines for naming events in a DDD context:

> - **Express Intent Clearly**:
>   - Choose names that clearly express the intent or meaning of the event.
>   - Use business language to name events, making them understandable to both technical and non-technical stakeholders.

> - **Use Past Tense**:
>   - Name events in the past tense, as events represent something that has already happened in the domain.
>   - For example, instead of "OrderCreated," use "OrderWasCreated."

> - **Avoid Technical Jargon**:
>   - Events should be named in a way that reflects the language of the business domain, avoiding technical jargon that might be unclear to non-technical stakeholders.

> - **Be Specific**:
>   - Be specific and precise in naming events to avoid ambiguity. Each event should represent a specific state change or occurrence in the domain.

> - **Consistency Across Bounded Contexts**:
>   - If your system involves multiple bounded contexts, strive for consistency in event naming across these contexts. This helps maintain a shared understanding of the system.

> - **Include Context Information**:
>   - Include relevant context information in the event names. This might include the aggregate or entity involved, the type of action, and any other relevant details.

> - **Reflect Business Processes**:
>   - Events often correspond to business processes. Reflect these processes in the event names to make it clear how the events fit into the overall flow of the business.

> - **Collaborate with Domain Experts**:
>   - Collaborate closely with domain experts to ensure that event names accurately reflect the language and concepts used in the business domain.

> - **Avoid Redundancy**:
>   - Avoid redundancy in event names. Each event should convey unique information, and unnecessary duplication should be minimized.

> - **Consider Event Semantics**:
>   - Think about the semantics of the events. Ensure that the names convey not just the what, but also the why and the impact of the event.

> - **Keep it Concise**:
>   - Strive for concise event names that convey the necessary information without unnecessary verbosity.

The key is to maintain a shared understanding of the domain among all stakeholders and to create a ubiquitous language that bridges the gap between technical and non-technical team members.

#### Some examples of event naming following the previous guidelines

**Express Intent Clearly:**

- **Good:** OrderPlaced
- **Avoid:** OrderUpdated

**Use Past Tense:**

- **Good:** PaymentProcessed
- **Avoid:** ProcessPayment

**Avoid Technical Jargon:**

- **Good:** ProductPurchased
- **Avoid:** ProductTransactionCompleted

**Be Specific:**

- **Good:** CustomerAddressUpdated
- **Avoid:** CustomerUpdated

**Consistency Across Bounded Contexts:**

- **Good:** InventoryItemDepleted
- **Good:** ProductOutOfStock
  - (Consistency in conveying stock-related events)

**Include Context Information:**

- **Good:** CartItemAdded
- **Good:** CartItemRemoved
  - (Includes context information about the shopping cart)

**Reflect Business Processes:**

- **Good:** OrderShipped
- **Good:** OrderDelivered
  - (Reflects the stages in the order fulfillment process)

**Collaborate with Domain Experts:**

- **Good:** CustomerAccountClosed
  - (Reflects the business language used by domain experts)

**Avoid Redundancy:**

- **Good:** UserLoggedIn
- **Avoid:** UserLoginSuccessful

**Consider Event Semantics:**

- **Good:** PromotionApplied
  - (Clearly conveys the impact of a promotion being applied)

**Keep it Concise:**

- **Good:** ReviewSubmitted
- **Avoid:** CustomerReviewHasBeenSubmittedSuccessfully

#### Examples for long-running operations

**Use Past Tense:**

- **Good:** PipettorAspirationCompleted
  - This clearly indicates that the aspiration process has been completed.

**Include Context Information:**

- **Good:** PipettorAspirationStarted
- **Good:** PipettorAspirationCompleted
- **Good:** PipettorAspirationFailed
  - These events include context about the specific phase of the workflow.

**Reflect Business Processes:**

- **Good:** LiquidAspirated
  - This event reflects the business process without specifying the actor (in this case, the pipettor).

**Avoid Redundancy:**

- **Good:** LiquidAspirated
- **Avoid:** PipettorAspirationEvent
  - The latter is redundant and adds unnecessary information.

**Consider Event Semantics:**

- **Good:** LiquidAspirated
  - This event conveys the essential information without being overly specific about the technical details.

## Guidelines

This section describes the guidelines to be followed in order to name events, commands and requests/responses (queries) in MolLab Instrument Software products.

### General Message naming guidelines

The following guidelines are applicable to all types of messages (events, commands, requests/responses):

`Format: (MUST Words) are required, (OPTIONAL Words) are optional`

- Names are written in **PascalCase**
  - ***Good**: ReagentCassetteLoaded, PostponeScheduledRuns, UnlockDrawer*
  - ***Avoid**: reagentCassetteLoaded, reagent_cassette_loaded, Reagent_Cassette_Loaded, etc...*
- Abbreviation are written in **PascalCase**.
  - ***Examples***: *Rfid, Mgp*
- Make names ***as specific and descriptive as possible***.
- Always provide examples for parameter of type string.
- Always provide a description for the message. Be as concise as possible, as the description will be used by third-party consumers of the data.

!!!Error Attention
    Avoid the use of the term `Finished` to indicate that a long-running operation has been successfully completed. The term `Finished` is ambiguous and it can be interpreted as `Completed` or `Failed`.

#### Commands

Commands represents **an intent to perform an action**. Commands are sent to a specific target (i.e. a specific software unit) and they are expected to immediately reply with either an Acknowledgement (ACK) or a Not Acknowledgement (NACK). Commands are used to shift responsibility from one software unit to another one. If the callee Acknowledges the command, it is expected that the command will be executed. If the callee does not Acknowledge the command, the caller must assume that the command will not be executed and, therefore, it must take the necessary actions to recover from the situation.

Commands:

- are written in an **imperative** form.
  - *DoSomething, PrepareSomething, etc...*
- if a command is not acknowledged, the sender must assume that the command was **not executed**.
- are not expected to be idempotent
  - this means they are not side-effect-free functions. In fact they are the opposite.
- a NACK Command is expected to be side-effect-free as no actions will be taken by the callee.
- **informs about completion** (successful or fail) of the operation **via event(s)**.

`Format`: (**MUST** Verb)(**MUST** Nouns,  pronouns,prepositions, entity)(**OPTIONAL** conjugated verb)(**OPTIONAL** noun)
`Examples`: *PrepareCassetteForPipetting, UnlockDrawer, StartRun, StopRun, PostponeScheduledRuns*
`Bad examples`: *AirPumpAspirateAir* because:

- The action (verb) must always be in the imperative form and at the beginning of the name.
- The target of the command must always be noun, pronoun or entity (*combined with prepositions if deemed useful*) used after the verb.
- Further details that helps providing meaning to the Command is appended at the end of the name (like a conjugated verb).
- `A name aligned with the guidelines would be` *AspirateAirWithAirPump*

##### Command Acknowledgement

Every command must **immediately (in a synchronous way)** reply with either an Acknowledgement (ACK) or a Not Acknowledgement (NACK). Acknowledgement indicates that the command can be accepted by the receiver and will be immediately executed. Acknowledgement **does not indicate that the command was executed**, it indicates that the command was accepted and **can** be executed. The following guidelines apply to the ACK/NACK messages:

`Format`: (**MUST** Command Name)Acknowledge
`Example`: PrepareCassetteForPipettingAcknowledge

- Use NACK for cases that the successfulness of execution can be checked before the actual execution of the command
  - For example: Trying to send a command that uses a redundant hardware component that is in error state or sending a command to an aggregate that is busy will, most probably, fail. In such cases (which can be quickly checked prior to executing the command) a NACK must be sent.
- Every command has **at least** the following acknowledges:
  - Success
  - Busy (if applicable)

!!!Hint Attention
    The list of acknowledges is not limited to the ones described above. Every command might have additional acknowledges depending on the specific implementation details and context of the command.

- In case of commands which start or stop background processes use a specific NACK for cases in which they are already running/stopped instead of busy. Examples of such background tasks could be: start/stop monitoring temperature, continuous collection of data in general. These backgrounds tasks are usually not blocking at Aggregate level and therefore the Aggregate **will not** report back `busy` while they are running.
- Examples:
  - AlreadyStarted
  - AlreadyCooling

#### Events

Events represents **a fact that occurred in the system**. Events are published by a specific software unit (and, more concretely by either an entity or process manager inside the software unit) and they can be consumed by any other software unit that is interested in them. Events are the result of a change of state of an entity or process manager. The publisher of an event is not aware of who is consuming the event and, therefore, it is not aware of what the consumer will do with the event.

Events:

- are **immutable**.
- specify which entity or process manager has changed its state.
  - Use of (composed) identities is encouraged. An identity is composed of one or more properties that uniquely identify an entity or process manager.
  - Since Process Managers are not always identifiable by themselves, the Identity might be a composition of properties that helps identifying the Process Manager.
    - Example: A "SampleTransferProcessManager" might be identified by the combination of the "RunId" and a fixed string that identifies the SampleTransfer as the type of process manager.

`Format`: (**OPTIONAL** conjugated verb)(**MUST** Nouns, pronouns, preposition, entity)(**OPTIONAL** Verb)(**MUST** simple past Verb)

##### Event representing 'simple' facts

Some events represent single fact. These events does not represent the starting/failure or progress of long-running operations. These events might represent the **completion** of long-running operations. These events are published as a result of a change of state of an entity.

`Example`: CassetteMoved, DoorOpened, UserLoggedIn, RunAborted, AirPumpAirAspirated.

##### Event representing the starting/failure or progress of long-running operations

When a Command is acknowledged it might start a long-running operation. In such cases, the software unit that is executing the command must publish an event indicating that the long-running operation has started.

`Format`: (**MUST** Command name with the nominalized verb moved to the end or process/workflow name)(**MUST** Started)
`Example`: ReagentCassetteTransferStarted, AirWithAirPumpAspirationStarted, RunExecutionStarted.

When a long-running operation fails, the unit that is executing the command must publish an event indicating that the long-running operation has failed.

`Format`: (**MUST** Command name with the nominalized verb moved to the end or process/workflow name)(**MUST** Failed)
`Example`: ReagentCassetteTransferFailed, AirWithAirPumpAspirationFailed, RunExecutionFailed.

##### Event representing the completion of long-running operations

When a long-running operation is completed, the unit that is executing the command must publish an event indicating that the long-running operation has completed. There are two options depending on the implementation details:

- When the long running operation is implemented by a Process Manager, the event indicating the completion of the long-running operation must be named as follows:
  - `Format`: (**MUST** Command name with the nominalized verb moved to the end or process/workflow name)(**MUST** Completed)
  - `Example`: ReagentCassetteTransferCompleted, AirWithAirPumpAspirationCompleted, RunExecutionCompleted.

**Rationale**: ProcessManagers purpose is to model long-running or complex operations where multiple aggregates (or other process managers) interoperate in order to achieve the expected goal. ProcessManagers are usually modeled as state machines, where transitions from one state to the next one are triggered by events. Entering a state is usually the trigger for a command to be executed. Because of this nature or process managers, their internal lifetime is better represented with Started, Failed and Completed events.

---

- When the long running operation is implemented by an Entity (usually an Aggregate Root), then the event indicating the completion of the long-running operation must be named as follows:
  - `Format`: (**OPTIONAL** conjugated verb)(**MUST** Nouns, pronouns, preposition, entity)(**MUST** simple past Verb)(**OPTIONAL** noun)
  - `Example`: ReagentCassetteTransferred, AirAspiratedWithAirPump, RunExecuted.

**Rationale**: Aggregate roots might provide operations that requires multiple entities within the Aggregate to be used. The Aggregate root is responsible, in this case, to coordinate the execution of the operations between the different entities. Because of this nature, the Aggregate root is the one that is responsible to publish the event indicating the completion of the long-running operation. Since Aggregates are the ones where the Domain concepts, business rules and invariants are properly modeled, the events published by the Aggregate root must reflect the Domain concepts and, therefore, the events published by the Aggregate root must be named as if they were a fact that occurred in the Domain.

#### Requests / Responses (Queries)

Requests/responses messages are used to request information from a specific software unit. The software unit receiving the request must reply with a response message. The response message must contain the requested information.

The following guidelines apply to Request messages:

- `Format`: (**MUST** Get)(**MUST** Nouns, pronouns, preposition, entity)
- `Example`: GetRunStatus, GetRunExecutionStatus, GetUnitStatus, GetLoggedInUser

The following guidelines apply to Response messages:

- If an entity, known in our domain, is requested the response should be:
  - `Format`: (**MUST** entity used in the request)
  - `Example`: RunStatus, RunExecutionStatus, UnitStatus, LoggedInUser

- Otherwise, the request should be (when a Nouns, pronouns, preposition is used for the request name):
  - `Format`: (**MUST** Nouns, pronouns, preposition)(**MUST** Response)
  - `Example`: GetRunStatusResponse, GetRunExecutionStatusResponse, GetUnitStatusResponse, GetLoggedInUserResponse

In this case a formal naming convention is used for the requests/responses messages. This is because the requests/responses messages are not part of the Domain and, therefore, they do not need to follow the Domain-Driven Design guidelines.

### Property naming guidelines

This section describes the guidelines to be followed in order to name the properties.

#### Boolean properties
The following guidelines apply to boolean properties of events:
- `Format`: (**MUST** Is)(**OPTIONAL** Nouns)(**MUST** Verb, Adjective)
- `Example`: IsActive, IsRunStartAllowed

### Message versioning

**First rule of message versioning, there is no message versioning!**
Once a unit publishing a message is released, it is no longer allowed to change the message schema. All messages sent over the message bus might be consumed by third party components (e.g. TellMe, DataHub). Therefor any change in the schema which has an influence on the serialization or delivery of a message is considered a breaking change for these components.

In case a released message schema needs to be adapted, the following steps need to be followed:
1. Create a copy of the schema and add a `_v2` suffix to the routing-key, title and schema filename.
2. Delete the old schema. The old message is no longer supposed to be published.
3. Adapt all affected `x-resulting-messages` and `x-triggered-by` accordingly.
4. Update the publisher of the message to use the new schema and publish the new message.
5. Update the consumers of the message to use the new schema and consume the new message.

Changes in the schema which have no influence on the serialization or delivery of a message are still allowed. 
E.g.
- Change description
- Change the following `x-` attributes:
  - `x-resulting-messages`
  - `x-triggered-by`
  
Note: Changes in the `x-routing-key`, `x-message-pattern`, `x-response-type` and `x-persistent` attributes are considered breaking.

## FAQ

### Why do we need to follow these guidelines?

- The guidelines are intended to provide a common language for all the software units in the instrument. This will help to understand the messages exchanged between software units.
- Homogeneity in the naming of messages will help to understand the messages exchanged between software units and between systems.
- Digital solutions making use of the data produced by the instrument will benefit from the homogeneity in the naming of messages.
- Eventually a common MolLab Event Catalog can emerge by following these guidelines.

### Why not using a fixed naming-conventions for event and commands instead of a domain-based approach as described in this document?

There were multiple proposals for fixed naming conventions for events and commands. Some examples of such proposals are:

- Events to be prefixed with `ev`
  - `<Prefix><Fact>`. hence 'RunAborted -> '**ev**RunAborted'.
- Events indicating the start of a long operation to be appended with `Started`.
  - `<Prefix><Fact>Started`. hence 'RunExecutionStarted -> '**ev**RunExecution**Started**'.
- Events representing successful completion to be appended with `Concluded`.
  - `<Prefix><Fact>Concluded` (concluded instead of 'completed', as it may not always 'complete'.) hence 'RunExecutionCompleted' -> '**ev**RunExecution**Concluded**'

- Commands to be prefixed with `cmd`.
- Commands triggering a lengthy operation to be form as: `<prefix>Start<Fact>`
  - hence StartRunExecution -> '**cmdStart**RunExecution’

Despite the fact that these naming conventions are easy to follow and would provide some benefits for IDEs (i.e. auto-completion), they do not provide any value for the business domain and they deviate from any DDD recommendation or practice followed in other industries. In this context, it is favored to follow a domain-based approach for naming events and commands, which is in line with the general ubiquitous language used in MolLab Instrument Software Systems.
