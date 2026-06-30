# Runtime Configuration

## Why This Document

One of the design goals of this framework is allowing different deployment
strategies without changing application code.

The runtime mode is therefore treated as an infrastructure configuration rather
than a programming model.

Applications should switch runtime modes simply by changing configuration.

---

# Runtime Modes

The framework currently defines three runtime modes.

| Mode | Description |
|------|-------------|
| native | Standard Spring Boot + Tomcat deployment |
| gateway | Transport protocol is converted into HTTP before entering Tomcat |
| bridge | Transport protocol enters Spring MVC directly |

The application itself remains identical in all three modes.

---

# Example Configuration

```yaml
spring:
  mvc:
    connector:
      enabled: true
      mode: gateway
```

or

```yaml
spring:
  mvc:
    connector:
      enabled: true
      mode: bridge
```

---

# Configuration Properties

## enabled

Whether Connector Runtime is enabled.

Default:

```yaml
enabled: false
```

When disabled, the application behaves exactly like a normal Spring Boot
application.

---

## mode

Selects the runtime mode.

Supported values:

### native

```
Client
    │
 HTTP
    │
Tomcat
    │
Spring MVC
```

No Connector components are activated.

---

### gateway

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

The Connector Runtime acts as a protocol gateway.

Tomcat continues to execute the complete Servlet lifecycle.

---

### bridge

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

The Connector Runtime dispatches requests directly into the Spring MVC runtime.

HTTP is no longer part of the request path.

---

# Runtime Selection

Different deployment scenarios benefit from different runtime modes.

The framework intentionally keeps runtime selection independent from
application development.

Business code should never need to know which runtime mode is active.

---

# Auto Configuration

The Connector Runtime should integrate with Spring Boot auto-configuration.

During startup the framework determines:

- whether Connector Runtime is enabled
- which runtime mode should be activated
- which infrastructure beans should be registered

Only the selected runtime implementation is created.

---

# Design Principles

## Configuration over Code

Changing runtime mode should require configuration changes only.

No Controller, Service or Repository should require modification.

---

## Infrastructure Transparency

Transport selection is an infrastructure concern.

Application developers should continue writing ordinary Spring MVC
applications.

---

## Progressive Adoption

Projects may adopt Connector Runtime gradually.

Example migration path:

```
Native
    │
    ▼
Gateway
    │
    ▼
Bridge
```

Each stage preserves the existing application while introducing a more advanced
runtime architecture.

---

# Future Extension

The runtime mode mechanism is designed to support additional Connector
implementations.

Possible future examples include:

- gRPC
- WebSocket
- Unix Domain Socket

Introducing new transports should not require changing the configuration model.

---

# Summary

Runtime mode is treated as an infrastructure capability.

Applications remain transport-independent.

Deployment strategy becomes a configuration decision instead of an architectural
rewrite.
