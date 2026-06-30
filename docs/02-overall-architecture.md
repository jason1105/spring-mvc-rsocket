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

The transport protocol belongs to the infrastructure layer.

---

## Runtime Modes

The framework supports multiple runtime modes.

Each mode shares the same Spring MVC programming model while using a different
strategy to introduce transport protocols.

The runtime mode is an infrastructure decision rather than an application
decision.

---

### Mode 0 - Native HTTP

This is the standard Spring Boot deployment model and serves as the reference
architecture for this project.

```
               Client
                  │
             HTTP / HTTPS
                  │
                  ▼
      Tomcat Connector (Coyote)
                  │
                  ▼
      Servlet Container Runtime
                  │
                  ▼
        DispatcherServlet
                  │
                  ▼
            Spring MVC
                  │
                  ▼
         Business Services
```

---

### Mode 1 - Gateway Mode

Gateway Mode converts a transport protocol into HTTP and forwards the request
to an existing Servlet container.

```
               Client
                  │
            RSocket / TCP
                  │
                  ▼
       Connector Gateway
      (RSocket → HTTP)
                  │
             HTTP Client
                  │
                  ▼
      Tomcat Connector (Coyote)
                  │
                  ▼
      Servlet Container Runtime
                  │
                  ▼
        DispatcherServlet
                  │
                  ▼
            Spring MVC
                  │
                  ▼
         Business Services
```

Characteristics:

- Existing Spring Boot application remains unchanged.
- Existing Tomcat deployment remains unchanged.
- Existing Servlet semantics are fully preserved.
- Suitable for rapid migration with minimal implementation cost.

---

### Mode 2 - Bridge Mode

Bridge Mode bypasses the HTTP connector and dispatches requests directly into
the Spring MVC runtime.

```
               Client
                  │
            RSocket / TCP
                  │
                  ▼
        Connector Bridge
                  │
                  ▼
     Servlet Runtime Adapter
                  │
                  ▼
        DispatcherServlet
                  │
                  ▼
            Spring MVC
                  │
                  ▼
         Business Services
```

Characteristics:

- No HTTP conversion.
- No Tomcat Connector.
- Maximum transport flexibility.
- Higher implementation complexity.

---

## Unified Architecture

```
                         Spring MVC
                              ▲
                              │
                  DispatcherServlet
                              ▲

