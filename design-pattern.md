# Design Pattern

- [Design Pattern](#design-pattern)
  - [Multitier/layer Architecture](#multitierlayer-architecture)
    - [Three-tier architecture](#three-tier-architecture)
  - [Service-Oriented Architecture (SOA)](#service-oriented-architecture-soa)
  - [Model-View-Controller (MVC)](#model-view-controller-mvc)
  - [Model-View-Presenter (MVP)](#model-view-presenter-mvp)
  - [Model–View–ViewModel (MVVM)](#modelviewviewmodel-mvvm)
    - [MVP vs MVVM](#mvp-vs-mvvm)
  - [Dependency Injection](#dependency-injection)
    - [Initializer Injection](#initializer-injection)
    - [Property Injection](#property-injection)
    - [Argument Injection](#argument-injection)
    - [Environment (or Context)](#environment-or-context)
    - [DI Framework](#di-framework)
    - [Instance Creation](#instance-creation)
  - [Key-Value Observation (KVO)](#key-value-observation-kvo)
  - [Singletons](#singletons)
  - [Circuit Breaker](#circuit-breaker)
  - [Builder Pattern](#builder-pattern)
  - [Factory Pattern](#factory-pattern)
  - [Communication Protocols](#communication-protocols)
    - [Server-Sent Events](#server-sent-events)
    - [REST](#rest)
      - [Authentication](#authentication)
    - [GraphQL](#graphql)
    - [gRPC](#grpc)
  - [Pagination](#pagination)
    - [Forward Pagination](#forward-pagination)
    - [Backward Pagination](#backward-pagination)
    - [Bidirectional Pagination](#bidirectional-pagination)
    - [Infinite Scroll](#infinite-scroll)
    - [Offset Pagination](#offset-pagination)
    - [Cursor Pagination](#cursor-pagination)
    - [**Cursor Pagination vs. Offset Pagination**](#cursor-pagination-vs-offset-pagination)

In software engineering, a design pattern describes a relatively small,
well-defined aspect (i.e. functionality) of a computer program in terms of
how to write the code.

Using a pattern is intended to leverage an existing concept rather than re-inventing
it. This can decrease the time to develop software and increase the quality of
the resulting program.

Notably, a pattern does not consist of a software artifact. Most development
resources that a programmer uses involve configuring the codebase to use an
artifact; for example a library. In contrast, to use a pattern, a programmer
writes code as described by the pattern. The result is unique every time even
though the result may be recognizable as based on the pattern.

Each pattern has the following elements:

- The **pattern name** is handle that is used to describe a design pattern.
- The **problem** describes when to apply the pattern. It explains the problem
and its context.
- The **solution** describes the elements that make up the design, their
relationships, responsibilities, and collaborations. The solution doesn't
describe the a particular concrete design or implementation, because a
pattern is like a template that can be applied in many different situations.
- The **consequences** are the results and tradeoffs of applying the pattern.
The consequences for software often concern space and trade-offs. Since reuse
is often a factor in object-oriented design, the consequences of a pattern
include its impact on a system’s flexibility, extensibility, or portability.

When choosing an architecture, must consider things such as:

- Project requirements: Consider whether the project will need scalability,
maintainability, testability, or flexibility in the future.
- Complexity and Simplicity: Strive for a balance between simplicity and
functionality. Overly complex patterns can make code more difficult to understand
and maintain.
- Testing Requirements: If unit testing is a priority, consider patterns that
promote testable code.
- Flexibility and Reusability: Some patterns provide better flexibility for code
reuse and modification. Patterns like Decorator or Strategy make it easier to
add or modify behaviors without extensive code changes.
- Team Experience and Consistency: Consider the team's familiarity with certain patterns.
- Scalability and Performance: Some patterns can be more demanding on resources,
so consider the performance needs of your app.

You can start by asking questions like:

- How dependency injection work
- How does testing work which goes hand-to-hand with dependency injection
- How does routing (for deep links) work
- Where does the business logic live

## Multitier/layer Architecture

In software engineering, multitier architecture (often referred to as n-tier
architecture) is a client–server architecture in which presentation,
application processing and data management functions are physically separated.
The most widespread use of multitier architecture is the three-tier architecture.

The following four are the most common layers:

- **Presentation** layer (a.k.a. UI layer, view layer, presentation tier in
multitier architecture)
- **Application** layer (a.k.a. service layer or GRASP Controller Layer)
- **Business** layer (a.k.a. business logic layer (BLL), domain logic layer)
- **Data** access layer (a.k.a. persistence layer, logging, networking, and other
services which are required to support a particular business layer)

The more usual convention is that the application layer (or service layer) is
considered a sublayer of the business layer, typically encapsulating the API
definition surfacing the supported business functionality.

### Three-tier architecture

Three-tier architecture is a client-server software architecture pattern in
which the user interface (presentation), functional process logic
("business rules"), computer data storage and data access are developed
and maintained as independent modules, most often on separate platforms.

Advantages of 3-Layer Architecture:

- **Separation of Concerns:** Each layer focuses on a specific responsibility.
- **Reusability:** Components of each layer can be reused across the application.
- **Maintainability:** Changes to one layer don’t typically affect others.
- **Scalability:** Easier to scale different parts of the system independently.

Disadvantages:

- **Overhead:** Can introduce unnecessary complexity for simple applications.
- **Latency:** Increases the number of interactions between layers, potentially
adding latency.

## Service-Oriented Architecture (SOA)

SOA organizes an application as a collection of loosely coupled services that
communicate over a network. Each service performs a specific business function
and interacts with others through well-defined APIs.

It's useful in cases such:

- Ideal for large, distributed systems with multiple teams.
- Enables integration of diverse technologies and platforms.

## Model-View-Controller (MVC)

It is a high-level pattern in that it concerns itself with the global
architecture of an application and classifies objects according to the general
roles they play in an application. It is also a compound pattern in that it
comprises several, more elemental patterns. The MVC design pattern considers
there to be three types of objects: model objects, view objects, and controller
objects<sup>[*](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html#//apple_ref/doc/uid/TP40010810-CH14-SW1)</sup>.

- Model: model objects encapsulate data and basic behaviors.
- View: view objects present information to the user.
- Controller: controller objects tie the model to the view.

Normally, the view implementation instantiates the concrete presenter object,
providing a reference to itself.

## Model-View-Presenter (MVP)

Model–view–presenter (MVP) is a derivation of the model–view–controller (MVC)
architectural pattern, and is used mostly for building user interfaces.

- The *model* is an interface defining the data to be displayed or otherwise acted
upon in the user interface.
- The *view* is a passive interface that displays data (the model) and routes user
commands (events) to the presenter to act upon that data.
- The *presenter* acts upon the model and the view. It retrieves data from
repositories (the model), and formats it for display in the view.

## Model–View–ViewModel (MVVM)

The Model-View-ViewModel (MVVM) design pattern to create a clear separation between
UI and business logic, resulting in modular, testable, and maintainable code.

- Model: represents the core data or business logic of the application. It contains
  the data structure and any operations or transformations related to the data.
  This layer should be isolated and independent of any UI code to keep business
  logic separate from presentation logic​.
- View: this layer includes the user interface, typically implemented through
SwiftUI views or UIKit components, depending on the framework. It doesn’t
directly interact with the Model. Instead, it relies on the ViewModel to mediate
data changes and interactions​.
- ViewModel: Acts as the intermediary between the Model and the View. The
ViewModel transforms data from the Model into a format that is easy for the View
to display. It also interprets user interactions from the View, updating the Model
as needed.

In MVVM, the responsibility for fetching data from the BE and mapping
it to models typically falls within a *service layer* or *repository layer* outside
of the Model-View-ViewModel triad. This layer interacts with the backend, parses
the data into model objects, and provides them to the ViewModel.

### MVP vs MVVM

Unlike the Presenter in MVP, the ViewModel in MVVM is state-driven and does not directly
"command" the View. It exposes observable properties (e.g., via Combine, RxSwift,
or KVO) that the View listens to for updates.

In MVVM, the View binds directly to these properties to reflect the state of the
data. It listens to changes in the ViewModel and updates itself automatically,
often with declarative bindings.

Here's a few advantages of MVVM compared to MVP:

- Decouples the View and the business logic further, making it easier to test the
ViewModel without the View.
- Provides a clean separation of responsibilities.
- Reactive bindings (e.g., via Combine) reduce the need for manual synchronization
between the View and ViewModel.

## Dependency Injection

Dependency injection in short means dependencies are handed instead of reaching
for them.

It is a programming technique in which an object or function
receives other objects or functions that it requires, as opposed to creating
them internally. Dependency injection aims to separate the concerns of
constructing objects and using them, leading to loosely coupled<sup>1</sup> programs.
The pattern ensures that an object or function that wants to use a given service
should not have to know how to construct those services. Instead, the receiving
"client" (object or function) is provided with its dependencies by external code
(an "injector"), which it is not aware of. Dependency injection makes
implicit dependencies explicit. In short, dependency injection is done to achieve:

- Isolation: the code for creating the dependency lives in one place.
- Reusability: goes hand-to-hand with isolation. More places can use a common
interface that hands over the dependency.
- Testability:
  - Tests can inject mocks in place of real dependency
  - Exercise type under test in isolation; so that your test isn't testing the
  entire codebase
  - Changes to other types shouldn't break your test

When picking an approach, note that there's no "one size fits all" solution.
Evaluate your needs, priorities and preferences. For instance, think about
decentralization, compile-time safety, thread safety or the incremental path.

Some of the common ways to do dependency injections:

<sup>1</sup> A loosely coupled system is one:

- in which components are weakly associated (have breakable relationships) with each
other, and thus changes in one component least affect existence or performance
of another component.
- in which each of its components has, or makes uses of, little or no knowledge
of the definitions of other separate components.

### Initializer Injection

It is simple and straightforward. The passed value would be immutable. But there's
a "superset" problem. Parents must have superset of all their descendants' dependencies.
Lastly, challenging for foundational dependencies (e.g., storage, networking,
and analytics). Lastly, static functions, protocols and extensions don't have initializer.

The compile-time safety means there's no need to deal with codes like this and
hope at that at runtime it won't crash:

```Swift
func resolve<T>(...) -> T?
```

### Property Injection

Same problems as [Initializer Injection](#initializer-injection), but it can work
for static members (but not protocol extensions). But the dependency now has to
be exposed, i.e., either internal or public. Lastly, sometimes it has to be either
optional or implicitly unwrapped optional.

### Argument Injection

This works for static functions, protocol extensions and non-OOP code as well.
But as API boilerplate (how many parameters are we going to pass into it) and
has the same "superset" problem that was explained in [Initializer Injection](#initializer-injection).

### Environment (or Context)

Like one singleton to rule them all right. This avoids the superset problem, and
has a minimal boilerplate. However, it doesn't scale nicely. It works for a few
things but when there's thousands of dependencies, it'll become a big environment.
Also, an environment that knows all about these types, those types can depend on
things that cause a circular import cycle in your modular structure.

```Swift
func update() {
  Environment.rideAPI.sendUpdate(...)
}
```

### DI Framework

These framework requires someone to work on them. But once it's built, i'll just
work. Handles circular dependencies, thread safety, compile-time safety, and
scoping like local scopes, graphs. singletons.

But it brings in some complexity and requires hundreds or thousands lines of code.
There's a hard learning curve. Often makes it harder to reason about the production
code.

Incremental adoption can be difficult and it's just one dependency to rule them all.

### Instance Creation

The goal is to separate business logic from instance creation responsibility.
This aligns well with the *Single Responsibility Principle*.

This approach is based on the principle provided by [Patrick Barry from Lyft](https://www.youtube.com/watch?v=dA9rGQRwHGs).
A DI system, at a high level, creates an instance like this:

```Swift
() -> T
```

However, this code doesn't state how to create an instance; so, let's add that:

```Swift
(() -> T) -> () -> T
```

 The first closure is actually creating the instance and returning a concrete
 type whereas the second one returns an interface. But then the compiler might
 return the same closure again. So, the compiler needs to know what type is
 expected to get back, which is the abstract interface:

 ```Swift
bind<T>(_ type: T.Type, to instantiator: @escaping () -> T) -> () -> T
 ```

## Key-Value Observation (KVO)

Key-value observing is a Cocoa programming pattern you use to notify objects about
changes to properties of other objects. It’s useful for communicating changes
between logically separated parts of your app—such as between models and views.
You can only use key-value observing with classes that inherit from `NSObject`.

Properties that are marked with both `@objc` and `dynamic`, can be observed using
KVO:

```Swift
class MyObjectToObserve: NSObject {
  @objc dynamic var myDate = NSDate(timeIntervalSince1970: 0)
  func updateDate() {
    myDate = myDate.addingTimeInterval(Double(2 << 30))
  }
}
```

An instance of an observer class manages information about changes made to one
or more properties.

```Swift
class MyObserver: NSObject {
  @objc var objectToObserve: MyObjectToObserve
  var observation: NSKeyValueObservation?
  
  init(object: MyObjectToObserve) {
    objectToObserve = object
    super.init()
    
    observation = observe(
      \.objectToObserve.myDate,
       options: [.old, .new]
    ) { object, change in
      print("OldValue: \(change.oldValue!), NewValue: \(change.newValue!)")
    }
  }
}
```

## Singletons

The Singleton is a design pattern that restricts a class to a single instance,
ensuring only one object of the class exists throughout the application. It’s
often used for shared resources like configurations, logging, network sessions,
or databases, where multiple instances could cause inconsistencies or conflicts.

Some cons of using singletons are:

- **Global State Dependency:** Global access can make testing and debugging
difficult, as parts of the code become dependent on shared state, leading to
tight coupling.
- **Limited Flexibility:** Since only one instance exists, replacing it for
testing or extending it for other purposes can be challenging.
- **Hidden Dependencies:** Singleton introduces hidden dependencies, as code
relying on a singleton may indirectly depend on the singleton’s state, making
code behavior less predictable.
- **Multithreading Issues:** Without proper thread-safety mechanisms, singletons
can cause race conditions in multithreaded contexts.

## Circuit Breaker

Circuit breaker is a design pattern used in software development. It is used
to detect failures and encapsulates the logic of preventing a failure from
constantly recurring, during maintenance, temporary external system failure
or unexpected system difficulties. It is commonly used in scenarios like
network requests or database calls.

## Builder Pattern

The builder pattern is a design pattern that provides a flexible solution
to various object creation problems in object-oriented programming.
The builder pattern separates the construction of a complex object from its representation.
The intent of the builder design pattern is to separate the construction of a
complex object from its representation. By doing so, the same construction
process can create different representations.

The builder design pattern solves problems like:

- How can a class (the same construction process) create different
representations of a complex object?
- How can a class that includes creating a complex object be simplified?

The builder design pattern describes how to solve such problems:

- Encapsulate creating and assembling the parts of a complex object in a
separate `Builder` object.
- A class delegates object creation to a `Builder` object instead of creating
the objects directly.

A class (the same construction process) can delegate to different `Builder`
objects to create different representations of a complex object.

Advantages of the builder pattern include:

- Allows you to vary a product's internal representation.
- Encapsulates code for construction and representation.
- Provides control over the steps of the construction process.

Disadvantages of the builder pattern include:

- A distinct ConcreteBuilder must be created for each type of product.
- Builder classes must be mutable.
- May hamper/complicate dependency injection.
- In many null-safe languages, the builder pattern defers
compile-time errors for unset fields to runtime.

## Factory Pattern

In object-oriented programming, the factory method pattern is a design pattern
that uses factory methods to deal with the problem of creating objects without
having to specify their exact classes. Rather than by calling a constructor,
this is accomplished by invoking a factory method to create an object.
Factory methods can be specified in an interface and implemented by subclasses
or implemented in a base class and optionally overridden by subclasses.

The factory method pattern relies on inheritance, as object creation is
delegated to subclasses that implement the factory method to create objects.
The pattern can also rely on the implementation of an interface.

## Communication Protocols

There's various protocols and technologies when it comes to establishing
a communication channel with the backend to send/receive data by hitting
the API endpoints.

### Server-Sent Events

Server-Sent Events (SSE) is a standard for unidirectional communication where a
server pushes real-time updates to a client over an HTTP connection.
It is part of the HTML5 specification and provides an efficient way for a
server to send event-driven data to the client.

The advantages of using SSE is that it can automatically reconnect when
the connection drops and it removes the need for a constant polling
(the server will notify the client when there's an update).
However, it doesn't support client-to-server communication, supports only
text (not data), and buffering is annoying, you can receive partial
packets and need to buffer it up until you know you've got a full event.

### REST

A REST API, also known as a RESTful API, is a simple, uniform interface
that is used to make data, content, algorithms, media, and other digital
resources available through web URLs.

Before REST, most developers had to deal with SOAP to integrate APIs.
SOAP was notorious for being complex to build, use, and debug.

REST operations are stateless. Request from client to server must contain
all of the information necessary so that the server can understand and
process it accordingly. The server can’t hold any information about the client state.

REST APIs operate over HTTP, which is request-response-based. They are not
inherently designed for real-time updates, requiring polling or long-polling
mechanisms to simulate real-time communication. This can lead to inefficiencies
compared to WebSockets, Server-Sent Events, or gRPC.

REST APIs are usually designed with fixed endpoints and responses,
which can lead to:

- **Over-fetching:** Clients receive more data than they need because
response may include unnecessary fields.
- **Under-fetching:** Clients make multiple requests to different endpoints to
gather all required data.
- **Scalability Issues**: As applications grow, managing numerous endpoints
for different entities and operations can become cumbersome. Also, new features
would need new endpoints.

#### Authentication

**HTTP Authentication:** HTTP provides authentication schemes for REST API implementation.
Two common schemes are:

- **Basic authentication:** HTTP basic authentication (BA) is a simple technique
for controlling access to web resources. It doesn’t require cookies, session
identifiers, or login pages. Instead, it uses standard fields in the HTTP header.
- **Bearer authentication:** Bearer authentication, also known as token authentication,
is an HTTP authentication scheme that involves the use of bearer tokens for security.

**API keys:** One way to authenticate REST APIs is with API keys.
When a client connects to a server for the first time, it is given a
unique identifier. This unique API key is then utilized for authentication on
every subsequent request to retrieve resources. It’s important to note that API
keys have security risks because they must be transmitted with each request and
can therefore be intercepted.

**OAuth:** OAuth is a security protocol that offers highly secure login access
to any system by combining passwords and tokens. The authorization process
starts with the server requesting a password, followed by an additional token
to complete the process.

REST is easy to learn, understand, and implement, easy to cache using built-in
HTTP caching mechanism. Also, there's loose coupling between client and server.
However, it's less efficient on mobile platforms since every request requires a
separate physical connection. It's schemaless, makes it hard to check data
validity on the client. It's stateless; so, needs extra functionality to maintain
a session. Lastly, additional overhead is needed (every request contains
contextual metadata and headers).

### GraphQL

GraphQL is a data query and manipulation language for APIs that allows a client
to specify what data it needs ("declarative data fetching"). A GraphQL server
can fetch data from separate sources for a single client query and present the
results in a unified graph. It is not tied to any specific database or storage engine.

A query language for working with API - allows clients to request data from
several resources using a single endpoint (instead of making multiple requests
in traditional RESTful apps).

**Pros:**

- Schema-based typed queries - clients can verify data integrity and format.
- Highly customizable - clients can request specific data and reduce the amount of
HTTP-traffic.
- Bi-directional communication with GraphQL Subscriptions (WebSocket based).

**Cons:**

- More complex backend implementation.
- "Leaky-abstraction" - clients become tightly coupled to the backend.
- The performance of a query is bound to the performance of the slowest service
on the backend (in case the response data is federated between multiple services).

### gRPC

gRPC is based on the client-server model, where a client can directly call
methods on a server application as if they were local methods. Communication
happens over HTTP/2, enabling features like multiplexing, bidirectional
streaming, and low-latency communication.

Some of the key features of gRPC are:

- Enables full-duplex communication and better performance compared to HTTP/1.1.
- Efficient serialization and deserialization.
- Supports various types of streaming (unary, server-side, client-side, and bidirectional).
- Supports multiple programming languages, making it suitable for cross-platform
systems.
- APIs are defined using Protobuf, ensuring a strict contract between client and
server.

Some of the use-cases of gRPC are:

- **Microservices Communication:** Ideal for low-latency, efficient communication
between services.
- **Real-Time Applications:** Bidirectional streaming for chat apps, gaming, or IoT.
- **Mobile and IoT:** Efficient communication where bandwidth is limited.
- **Polyglot Environments:** Teams working with different programming languages.

Limitations of gRPC are:

- **Steeper Learning Curve:** Requires understanding of Protobuf and streaming concepts.
- **Debugging:** Harder to debug compared to REST (binary payloads are less human-readable).
- **Limited Browser Support:** gRPC-Web is required for frontend use, which adds
complexity.

## Pagination

Endpoints that return a list of entities must support pagination.
Without pagination, a single request could return a huge amount of
results causing excessive network and memory usage.

### Forward Pagination

Loads new messages as the user scrolls downward in the chat view.
Can use techniques like polling or WebSockets to get new messages and
append them to the end of the list.

Some of the use-cases are:

- For channels or public groups with high traffic where the focus is on
keeping up with the conversation.
- For live chat scenarios, where new messages are expected frequently.

Some of the challenges are:

- Ensuring seamless scrolling when appending new messages to the bottom.
- Handling real-time updates alongside paginated loading.

### Backward Pagination

Loads older messages as the user scrolls upward in the chat view.
Need to fetch older messages from the server when the user reaches the top of the
current message list.

Some of the use-cases are:

- When a user wants to view message history.
- Most common in private chats or low-traffic public channels where users may
want to reference past conversations.

Some of the challenges:

- Maintaining the user's scroll position after older messages are loaded.
- Balancing the batch size to prevent overwhelming the UI.

### Bidirectional Pagination

Supports both forward and backward pagination. Need to preload messages in both
directions with APIs that support offsets or
cursors for bidirectional data loading.

Some of the use-cases are:

- For group chats where users join at an arbitrary point in the conversation
and might scroll both forward and backward.
- When implementing features like "jump to a specific message" or "unread messages."

Some of the challenges are:

- Requires careful handling of scroll positions when adding data to both ends
of the list.

### Infinite Scroll

Messages are dynamically loaded in either direction as the user scrolls,
creating an "infinite" conversation history.

Some of the use-cases are:

- For apps with extensive message history (e.g., Slack or Discord-like apps).
- To provide a seamless user experience when browsing through large message sets.

Some of the challenges are:

- Requires efficient caching and memory management to avoid performance degradation.
- Handling scroll offsets carefully to ensure smooth navigation.

### Offset Pagination

Provides a limit and an offset query parameters. Example: GET `/feed?offset=100&limit=20`.

Pros:

- easiest to implement - the request parameters can be passed directly to a SQL query.
- stateless on the server.

Cons:

- bad performance on large offset values (the database needs to skip offset
rows before returning the paginated result).
- inconsistent when adding new rows into the database (Page Drift).

### Cursor Pagination

The cursor pagination utilizes a pointer that refers to a specific database record.
This approach is especially useful when dealing with large datasets or
real-time data that might change frequently.

### **Cursor Pagination vs. Offset Pagination**

| **Aspect** | **Cursor Pagination** | **Offset Pagination** |
|-|-|-|
| **Example** | `feed?after_id=p1b2&limit=10` | `feed?after_page=2&limit=10` |
| **Navigation** | Cursor-based | Offset-based|
| **Performance** | Efficient for large datasets| Slower with larger datasets |
| **Dynamic Data** | Handles real-time updates gracefully | May skip or duplicate data |
| **Complexity** | More complex to implement | Simple to implement |
| **Random Access** | Not possible | Easy to jump to any page |

Some of the use-cases are:

- Real-time applications (e.g., chat, news feeds).
- Large datasets where offset pagination becomes inefficient.
- Systems that frequently update or delete data, causing inconsistencies
in offset-based pagination.

Cursor-based pagination comes with [tradeoffs](https://slack.engineering/evolving-api-pagination-at-slack/):

- The cursor must be based on a unique, sequential column (or columns)
in the source table.
- There is no concept of the total number of pages or results in the set.
- The client can’t jump to a specific page.
