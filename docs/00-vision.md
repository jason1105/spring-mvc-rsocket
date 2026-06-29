# Spring MVC Connector Framework Vision

## 1. Core Idea

This project explores a fundamental architectural shift:

> Decouple Spring MVC from HTTP by introducing a pluggable Connector SPI.

Spring MVC should not be bound to HTTP/Tomcat.
Instead, HTTP is just one implementation of a Connector.

## 2. Architecture Philosophy

The system is divided into three layers:

### 2.1 Connector Layer (Transport Abstraction)
- HTTP (Tomcat / Jetty)
- RSocket (current PoC target)
- gRPC (future)
- TCP raw (future)

This layer is responsible for:
- protocol parsing
- request framing
- stream handling

### 2.2 Servlet Semantics Layer
- HttpServletRequest / Response abstraction
- FilterChain execution model
- ThreadLocal context propagation

### 2.3 Spring MVC Layer (unchanged)
- DispatcherServlet
- HandlerMapping
- Controller
- AOP / Security / Validation

## 3. Key Principle

> MVC must remain unchanged. All adaptation happens below DispatcherServlet.

## 4. RSocket Role

RSocket is NOT the core of the system.

It is only:
> the first implementation of a Connector.

## 5. Non-Goals

- Do NOT rewrite Spring MVC
- Do NOT replace DispatcherServlet
- Do NOT introduce new programming model (@MessageMapping replacement)

The goal is reuse, not replacement.

## 6. Design Constraint

All existing Spring MVC applications must run without modification.


