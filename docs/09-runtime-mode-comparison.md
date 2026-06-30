# Runtime Mode Comparison

## Why This Document

Connector Runtime supports multiple runtime modes.

Each mode targets a different balance between implementation complexity,
compatibility and runtime performance.

There is no universally superior mode.

Choosing a runtime mode is an architectural decision.

---

# Comparison Overview

| Capability | Native HTTP | Gateway Mode | Bridge Mode |
|------------|------------|--------------|-------------|
| Transport | HTTP | RSocket → HTTP | RSocket |
| Tomcat Connector | ✔ | ✔ | ✘ |
| Servlet Container | ✔ | ✔ | Partial (Adapter) |
| DispatcherServlet | ✔ | ✔ | ✔ |
| Spring MVC | ✔ | ✔ | ✔ |
| Controller Reuse | ✔ | ✔ | ✔ |
| Spring Security | Native | Native | Adapter |
| Filter Chain | Native | Native | Adapter |
| Servlet Semantics | Native | Native | Compatible Target |
| Upload / Download | Multipart | Multipart / Stream | Native Stream |
| Streaming | Limited | Limited | Native |
| Implementation Complexity | Very Low | Low | High |
| Runtime Performance | Baseline | Good | Best |

---

# Native HTTP

Native HTTP is the reference deployment model provided by Spring Boot.

Advantages:

- Mature ecosystem
- Full Servlet compatibility
- Lowest operational risk

Limitations:

- HTTP is mandatory
- Desktop protocols require additional adaptation

---

# Gateway Mode

Gateway Mode introduces a protocol gateway while preserving the existing
Servlet runtime.

Advantages:

- Existing Spring Boot applications remain unchanged.
- Existing Tomcat deployment remains unchanged.
- Existing Filters and Spring Security continue to work without adaptation.
- Lowest migration cost.

Limitations:

- Every request is translated into HTTP.
- HTTP remains part of the execution path.
- Additional network hop.

Gateway Mode is recommended when compatibility is more important than absolute
performance.

---

# Bridge Mode

Bridge Mode removes the HTTP connector from the request path.

Advantages:

- Native transport integration.
- No HTTP conversion.
- Better throughput for long-lived connections.
- Better support for streaming workloads.

Limitations:

- More complex implementation.
- Servlet semantics must be adapted.
- Infrastructure compatibility requires additional verification.

Bridge Mode is recommended when transport efficiency and long-term architecture
are more important than implementation simplicity.

---

# Recommended Scenarios

## Native HTTP

- Traditional Web applications
- Browser-based systems

---

## Gateway Mode

- Existing enterprise systems
- Low-risk migration
- Regulatory environments
- Projects prioritizing compatibility

---

## Bridge Mode

- Desktop applications
- High-concurrency systems
- Streaming workloads
- Long-term platform evolution

---

# Migration Path

The three runtime modes are intentionally designed to support progressive
adoption.

```
Native HTTP
       │
       ▼
 Gateway Mode
       │
       ▼
 Bridge Mode
```

Applications should not require business code changes during migration.

Only infrastructure configuration changes between stages.

---

# Summary

Native HTTP, Gateway Mode and Bridge Mode are complementary deployment
strategies.

They share the same Spring MVC programming model while optimizing different
engineering goals.

The runtime mode should therefore be selected according to project
requirements rather than framework capability.
