# Servlet Container

## Why This Document

One of the biggest misconceptions when working with Spring MVC is treating
Tomcat as an HTTP server.

In reality, HTTP is only a small part of its responsibility.

The real responsibility of Tomcat is providing a runtime environment capable of
executing Servlet applications.

Understanding this runtime boundary is the foundation of this project.

---

# What Is a Servlet Container?

A Servlet Container is a runtime responsible for converting network requests
into Java application execution.

It is responsible for:

- accepting network connections
- parsing HTTP requests
- creating Servlet request objects
- executing the Filter chain
- invoking Servlets
- managing the request lifecycle
- writing responses back to the client

Spring MVC is only one application running inside this runtime.

---

# Runtime View

```
Client
    │
    ▼
Servlet Container
    │
    ▼
Spring MVC
    │
    ▼
Business Logic
```

The Servlet Container owns the runtime.

Spring MVC owns the business execution model.

---

# Tomcat Architecture

Tomcat can be viewed as five logical layers.

```
┌─────────────────────────────┐
│ Protocol Layer              │
│ HTTP / HTTPS / AJP          │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│ Connector (Coyote)          │
│ Socket → Servlet Request    │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│ Catalina Container          │
│ Engine / Host / Context     │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│ Servlet Pipeline            │
│ Filter → Servlet            │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│ Spring MVC                  │
└─────────────────────────────┘
```

Although often discussed together, these layers have different responsibilities.

---

# Request Lifecycle

```
TCP Socket
      │
      ▼
HTTP Parsing
      │
      ▼
HttpServletRequest
      │
      ▼
Filter Chain
      │
      ▼
DispatcherServlet
      │
      ▼
Controller
      │
      ▼
Business Logic
      │
      ▼
HttpServletResponse
      │
      ▼
Socket Write
```

Notice that Spring MVC only starts after the Servlet request has already been
created.

Everything before DispatcherServlet belongs to the Servlet runtime.

---

# Responsibilities

## Connector

The Connector is responsible for:

- accepting sockets
- protocol parsing
- request creation
- response writing

The Connector knows nothing about Spring MVC.

---

## Servlet Container

The Servlet Container is responsible for:

- request lifecycle
- Filter execution
- Servlet invocation
- request context management

It knows nothing about business logic.

---

## Spring MVC

Spring MVC is responsible for:

- routing
- parameter binding
- validation
- controller execution
- exception handling

Spring MVC does not manage sockets.

Spring MVC does not parse HTTP.

Spring MVC assumes that a valid Servlet runtime already exists.

---

# Mapping to This Project

This project intentionally preserves the Spring MVC runtime.

Only the transport entry point changes.

```
Traditional

HTTP
    │
Tomcat Connector
    │
Servlet Runtime
    │
Spring MVC


This Project

RSocket
    │
Protocol Adapter
    │
Servlet Runtime
    │
Spring MVC
```

DispatcherServlet remains the execution boundary.

Controllers remain unchanged.

Business logic remains unchanged.

---

# Architectural Boundary

The project deliberately chooses DispatcherServlet as the architectural
boundary.

Everything below DispatcherServlet belongs to infrastructure.

Everything above DispatcherServlet belongs to the application.

This separation minimizes migration cost while maximizing compatibility with
existing Spring MVC applications.

---

# Key Observation

The purpose of this project is not replacing Spring MVC.

The purpose is replacing the transport entry point.

Spring MVC should continue to execute exactly as before.

Only the way requests enter the runtime changes.

---

# Summary

Tomcat is much more than an HTTP server.

