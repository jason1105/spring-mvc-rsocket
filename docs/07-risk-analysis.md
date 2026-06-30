# Risk Analysis

## Why This Document

The purpose of this project is to preserve the Spring MVC programming model
while introducing a completely different transport protocol.

Although this significantly reduces migration cost, it also introduces several
architectural risks that must be explicitly identified.

This document records the known risks discovered during the architecture
design phase.

---

# Risk Classification

Current risks are grouped into four categories:

- Servlet Semantics
- Spring Runtime Compatibility
- Threading Model
- Future Evolution

---

# Risk 1

## Servlet Request Semantics

The largest technical challenge is not request routing.

It is faithfully reproducing the execution semantics expected by Spring MVC.

A Servlet request is much more than:

- URI
- Headers
- Parameters
- Request Body

Spring MVC and its surrounding ecosystem may depend on behavioral semantics,
including:

- request attributes
- request lifecycle
- request dispatcher behavior
- locale information
- servlet context

The project should preserve these semantics whenever practical.

---

# Risk 2

## Filter Chain Compatibility

Spring Security and many infrastructure components execute inside the Servlet
Filter chain.

Typical examples include:

- Spring Security
- Logging
- Tracing
- Metrics
- CORS
- Character Encoding

If the request bypasses the Filter chain, these components may stop working or
behave inconsistently.

For this reason the Filter chain should remain part of the request execution
pipeline whenever possible.

---

# Risk 3

## DispatcherServlet Boundary

Calling Controller methods directly appears attractive because it seems simple.

However this bypasses a significant amount of Spring MVC infrastructure.

Examples include:

- HandlerMapping
- HandlerAdapter
- ArgumentResolver
- MessageConverter
- ExceptionResolver
- ControllerAdvice

Bypassing DispatcherServlet effectively creates a second MVC runtime.

This project explicitly rejects that approach.

---

# Risk 4

## ThreadLocal Context

Many Spring components rely on ThreadLocal storage.

Examples include:

- RequestContextHolder
- SecurityContextHolder
- LocaleContextHolder
- TransactionSynchronizationManager

If request execution jumps between worker threads,
these contexts may become inconsistent.

The current architecture therefore adopts a single worker thread execution
model after leaving the EventLoop.

---

# Risk 5

## Blocking Business Logic

Most existing Spring MVC applications use:

- JDBC
- Hibernate
- MyBatis
- Blocking Redis clients
- Blocking RPC frameworks

Blocking operations are expected.

Executing them on the networking EventLoop would dramatically reduce
throughput.

The architecture therefore introduces an explicit Business Executor.

---

# Risk 6

## Multipart Compatibility

Traditional Spring MVC applications frequently rely on Multipart support.

Multipart is deeply integrated with:

- HTTP
- Servlet parsing
- request boundaries

The current architecture prefers transport-native streaming.

Compatibility with existing Multipart-based applications requires additional
evaluation.

---

# Risk 7

## Async Servlet Features

Spring MVC supports asynchronous execution models such as:

- Callable
- DeferredResult
- WebAsyncManager

These features depend on Servlet asynchronous processing.

The initial implementation does not attempt to provide full compatibility.

This topic should be evaluated independently after the synchronous runtime
becomes stable.

---

# Risk 8

## Internal Spring APIs

The implementation should avoid depending on internal Spring Framework
implementation classes whenever possible.

Examples include:

- package-private classes
- undocumented extension points
- internal lifecycle callbacks

Doing so would increase maintenance cost when upgrading Spring Framework.

Preference should always be given to stable public APIs.

---

# Compatibility Goals

The following Spring MVC features are expected to remain compatible:

- @RestController
- @Controller
- @RequestMapping
- Validation
- AOP
- Spring Security
- Exception Handling
- Interceptors

Maintaining compatibility with these features is a primary design objective.

---

# Explicit Non-Goals

The project does not currently attempt to support every Servlet feature.

Examples include:

- HTTP Upgrade
- Servlet Push
- JSP
- Servlet Include / Forward

These features are unrelated to the primary project goals.

---

# Incremental Strategy

Rather than implementing every Servlet feature immediately,
the project follows an incremental compatibility strategy.

Phase 1

- synchronous request execution
- Spring MVC compatibility
- Security compatibility

Phase 2

- upload and download
- streaming

Phase 3

- advanced Servlet compatibility
- asynchronous execution

This staged approach reduces technical risk while allowing early validation of
the architecture.

---

# Summary

The largest challenge of this project is not introducing RSocket.

The real challenge is preserving the execution semantics expected by the Spring
MVC ecosystem.

Every architectural decision should therefore prioritize compatibility over
novelty.

The goal is for existing applications to migrate with minimal code changes
while retaining the behavior developers already expect from Spring MVC.
