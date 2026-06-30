# RSocket MVC Bridge

## Why This Document

The most important design decision of this project is determining where a
non-HTTP request should enter the Spring MVC runtime.

Choosing the wrong boundary would either:

- duplicate the existing Spring MVC infrastructure, or
- tightly couple the implementation to Spring internals.

The Bridge is introduced to isolate transport adaptation from application
execution.

---

# Problem Statement

Spring MVC applications are traditionally executed inside a Servlet container.

Incoming requests follow this path:

```
HTTP
    │
Servlet Container
    │
DispatcherServlet
    │
Controller
```

In this project, incoming requests originate from RSocket instead of HTTP.

The challenge is introducing a new transport without changing the existing
application.

---

# Design Goal

The Bridge has only one purpose:

> Make an RSocket request appear as a normal Servlet request.

Everything after DispatcherServlet should execute exactly as before.

Controllers should never know whether the request originated from HTTP or
RSocket.

---

# Position in the Architecture

```
JavaFX
    │
RSocket
    │
RSocket MVC Bridge
    │
DispatcherServlet
    │
Controller
    │
Service
```

The Bridge is the only component aware of both:

- transport protocol
- Servlet runtime

No other layer should depend on both worlds.

---

# Responsibilities

The Bridge is responsible for:

- receiving transport requests
- decoding protocol payloads
- creating request context
- preparing Servlet request semantics
- dispatching requests into Spring MVC
- converting responses back into transport messages

The Bridge is an infrastructure component.

It does not contain business logic.

---

# What the Bridge Does NOT Do

The Bridge is intentionally limited.

It must not:

- implement business logic
- replace DispatcherServlet
- replace Spring MVC routing
- replace parameter binding
- replace validation
- replace exception handling

Those responsibilities already belong to Spring MVC.

---

# Why Not Call Controllers Directly?

Calling Controllers directly appears simple but would bypass a significant
portion of the Spring MVC runtime.

The following infrastructure would be skipped:

- HandlerMapping
- HandlerAdapter
- ArgumentResolver
- MessageConverter
- ControllerAdvice
- ExceptionResolver
- Validation
- Security integration

This would create a second execution model that behaves differently from HTTP.

That approach is intentionally rejected.

---

# Why DispatcherServlet?

DispatcherServlet already represents the execution boundary of Spring MVC.

Everything required for business execution begins here.

By dispatching requests through DispatcherServlet:

- existing Controllers remain unchanged
- existing infrastructure continues to work
- migration cost stays minimal

For this reason DispatcherServlet is selected as the runtime boundary.

---

# Request Flow

```
JavaFX
    │
RSocket Frame
    │
Bridge
    │
Servlet Request Context
    │
DispatcherServlet
    │
HandlerMapping
    │
HandlerAdapter
    │
Controller
    │
Business Service
    │
Response
    │
Bridge
    │
RSocket Frame
```

Only the entry point changes.

Everything inside Spring MVC remains unchanged.

---

# Design Constraints

The Bridge must preserve existing Spring MVC semantics.

In particular:

- Controller mapping
- Parameter binding
- Validation
- Exception handling
- Security
- AOP

Applications should behave identically regardless of transport protocol.

---

# Compatibility

The Bridge is designed to maximize compatibility with existing applications.

Existing projects should continue to use:

- @RestController
- @RequestMapping
- @PostMapping
- @Validated
- @ControllerAdvice
- Spring Security

without modification.

Migration should not require introducing another programming model.

---

# Current Scope

The current Proof of Concept focuses on synchronous request execution.

Support for advanced Servlet features such as asynchronous request processing
will be evaluated separately.

This incremental approach keeps the initial implementation focused and reduces
technical risk.

---

# Summary

The RSocket MVC Bridge is the architectural boundary between transport protocols
and the Spring MVC runtime.

Its responsibility is adaptation rather than execution.

Business execution continues to belong entirely to Spring MVC.

This separation allows new transport protocols to be introduced while preserving
the existing application and minimizing migration cost.
