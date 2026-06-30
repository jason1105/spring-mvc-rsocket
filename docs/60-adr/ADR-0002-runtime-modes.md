# ADR-0002

# Support Multiple Connector Runtime Modes

## Status

Accepted

---

## Context

The primary objective of this project is preserving the Spring MVC programming
model while allowing applications to communicate through transport protocols
other than HTTP.

During the architecture design process, two different implementation
strategies emerged.

The first strategy forwards requests to the existing Servlet container.

The second strategy bypasses the Servlet connector and dispatches requests
directly into the Spring MVC runtime.

Both approaches satisfy different engineering goals.

Instead of selecting one and discarding the other, the framework chooses to
support both as first-class runtime modes.

---

## Decision

The framework defines three runtime modes.

### Native HTTP

```
Client
    │
 HTTP
    │
Tomcat
    │
Spring MVC
```

This is the reference deployment model provided by Spring Boot.

---

### Gateway Mode

```
Client
    │
RSocket
    │
Gateway
    │
HTTP
    │
Tomcat
    │
Spring MVC
```

Gateway Mode converts transport protocols into HTTP and delegates request
processing to the existing Servlet container.

---

### Bridge Mode

```
Client
    │
RSocket
    │
Bridge
    │
DispatcherServlet
    │
Spring MVC
```

Bridge Mode adapts transport protocols directly into the Spring MVC runtime.

The HTTP connector is no longer part of the execution path.

---

## Design Rationale

Supporting multiple runtime modes provides several advantages.

### Preserve Existing Investments

Gateway Mode allows existing Spring Boot applications to adopt new transport
protocols without modifying deployment architecture.

Tomcat, Servlet Filters and operational tooling remain unchanged.

---

### Enable Long-Term Evolution

Bridge Mode removes the dependency on the HTTP connector and enables a
transport-independent runtime architecture.

It becomes the foundation for future Connector implementations.

---

### Separate Business Architecture from Runtime Architecture

Application developers should not redesign business code because deployment
strategy changes.

Controllers, Services and business workflows remain identical regardless of
runtime mode.

---

## Alternatives Considered

### Alternative 1

Support only Gateway Mode.

Advantages:

- Simple implementation
- Lowest compatibility risk

Disadvantages:

- HTTP remains mandatory
- Additional protocol conversion
- Limited long-term evolution

Rejected.

---

### Alternative 2

Support only Bridge Mode.

Advantages:

- Cleaner architecture
- Better performance
- Transport independence

Disadvantages:

- Higher implementation complexity
- Higher migration risk
- Larger compatibility surface

Rejected.

---

## Consequences

Connector Runtime evolves from a single implementation into a runtime
architecture.

Transport protocols become infrastructure concerns rather than application
concerns.

The application layer continues to execute the same Spring MVC programming
model regardless of runtime mode.

---

## Architectural Principles

### Principle 1

Runtime mode is selected by configuration.

Application code should not change.

---

### Principle 2

Gateway Mode and Bridge Mode are both first-class runtime modes.

Neither mode is considered temporary, experimental or transitional.

---

### Principle 3

Gateway Mode optimizes operational compatibility.

Bridge Mode optimizes runtime architecture.

Both are valid engineering choices.

---

### Principle 4

Future Connector implementations should integrate into the same runtime model
without introducing another application programming model.

---

## Future Evolution

The runtime mode architecture allows future Connector implementations,
including but not limited to:

- gRPC
- WebSocket
- Unix Domain Socket
- Named Pipe

New transport protocols should become additional infrastructure modules rather
than new application frameworks.

---

## Summary

The framework deliberately supports multiple runtime modes because different
projects optimize different engineering goals.

Gateway Mode emphasizes compatibility, operational simplicity and migration
cost.

Bridge Mode emphasizes transport independence, architectural evolution and
runtime performance.

The coexistence of both modes provides a practical migration path while
establishing a long-term transport-independent architecture for Spring MVC.
