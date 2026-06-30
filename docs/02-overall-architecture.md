# Overall Architecture

## Background

The existing application is built on:

- Spring Boot 3
- Spring MVC
- Tomcat
- React

The frontend will migrate from React to JavaFX.

The communication protocol will migrate from HTTP to TCP.

The primary objective of this project is to preserve the existing Spring MVC
application while introducing a new transport protocol.

Business logic should remain unchanged throughout the migration.

---

## Design Objectives

The project is designed around the following objectives:

- Preserve the existing Spring MVC runtime.
- Reuse all existing Controllers.
- Reuse Spring Security, AOP and Validation.
- Avoid introducing another endpoint programming model.
- Minimize migration cost.

The transport protocol should become an infrastructure concern rather than an
application concern.

---

## High-Level Architecture

```
                +--------------------+
                |    JavaFX Client   |
                +----------+---------+
                           |
                    RSocket (TCP)
                           |
                +----------v---------+
                | Protocol Adapter   |
                +----------+---------+
                           |
                  Servlet Semantics
                           |
                +----------v---------+
                | DispatcherServlet  |
                +----------+---------+
                           |
                +----------v---------+
                |    Spring MVC      |
                +----------+---------+
                           |
                +----------v---------+
                |  Business Services |
                +----------+---------+
                           |
                     Database / RPC
```

---

## Layer Responsibilities

### JavaFX Client

Responsible for:

- User interaction
- Desktop experience
- Communication with the backend

---

### RSocket

Responsible for:

- Long-lived TCP connections
- Request/Response messaging
- Streaming
- Flow control

Within this project, RSocket is treated purely as the transport protocol.

---

### Protocol Adapter

The adapter acts as the boundary between the transport layer and the Spring MVC
runtime.

Responsibilities include:

- Decode transport messages
- Build request context
- Adapt transport semantics to Servlet semantics
- Dispatch requests into Spring MVC

The adapter must never contain business logic.

---

### Spring MVC

Spring MVC remains completely unchanged.

It continues to provide:

- Request Mapping
- Parameter Binding
- Validation
- Exception Handling
- Security
- AOP

The existing application should not be aware of the newly introduced transport.

---

## Design Principles

### Preserve Existing Runtime

Spring MVC is considered an existing runtime rather than something to be
rewritten.

The project adapts incoming requests instead of modifying Spring MVC.

### Single Business Implementation

Business logic should exist only once.

Different transport protocols must not require duplicated Controllers.

### Transport Independence

Business code should not depend on whether requests originate from:

- HTTP
- RSocket
- Future transport implementations

