# Thread Model

## Why This Document

Spring MVC applications are traditionally executed on a dedicated request
thread provided by the Servlet container.

RSocket, however, is built on a reactive networking model where a small number
of EventLoop threads are responsible for serving many concurrent connections.

Understanding the boundary between these two threading models is essential to
the architecture of this project.

---

# Two Different Concurrency Models

## Servlet Model

```
Request
    │
Servlet Thread
    │
Controller
    │
Database
```

One request occupies one worker thread until completion.

This matches the execution model of most existing Spring MVC applications.

---

## RSocket / Netty Model

```
Socket
    │
EventLoop
    │
Many Connections
```

An EventLoop is designed for:

- socket IO
- frame parsing
- scheduling

It is not designed for blocking operations.

---

# Existing Application Characteristics

This project targets existing Spring MVC applications.

Typical characteristics include:

- JDBC
- JPA / Hibernate
- MyBatis
- Redis clients
- Blocking RPC frameworks

These applications are fundamentally blocking.

Changing them into fully reactive applications is outside the scope of this
project.

---

# Design Principle

The transport layer may be reactive.

The business layer remains blocking.

Therefore a clear thread boundary must exist between them.

---

# Thread Architecture

```
                RSocket Connection
                        │
                Netty EventLoop
                        │
                        ▼
          ┌─────────────────────────┐
          │ Thread Switch           │
          └────────────┬────────────┘
                       │
                       ▼
            Business Executor Pool
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
                    Database
```

Only one thread switch should occur.

Everything inside Spring MVC executes on the same worker thread.

---

# EventLoop Responsibilities

The EventLoop is responsible only for:

- accepting frames
- decoding payloads
- scheduling execution
- writing responses

The EventLoop must never perform:

- database access
- business computation
- blocking IO

Keeping the EventLoop lightweight maximizes throughput for all connections.

---

# Business Executor Responsibilities

The Business Executor owns the complete request execution.

Responsibilities include:

- Filter execution
- DispatcherServlet invocation
- Controller execution
- Service execution
- Database access

The request remains on the same worker thread until completion.

---

# Why Only One Thread Switch?

Many Spring components rely on ThreadLocal state.

Examples include:

- RequestContextHolder
- SecurityContextHolder
- LocaleContextHolder
- Transaction synchronization

Moving execution between multiple worker threads complicates context
propagation and increases implementation complexity.

A single thread switch keeps the runtime consistent with the traditional
Servlet execution model.

---

# Request Execution

```
EventLoop
      │
      ▼
Business Executor
      │
      ▼
Filter
      │
      ▼
DispatcherServlet
      │
      ▼
Controller
      │
      ▼
Repository
      │
      ▼
Database
      │
      ▼
Response
      │
      ▼
EventLoop
```

From the perspective of Spring MVC, execution remains entirely synchronous.

---

# Blocking Operations

Blocking operations are expected.

Examples include:

- SQL execution
- File access
- Legacy RPC frameworks

These operations are intentionally isolated from the EventLoop by the Business
Executor.

This preserves the scalability of the networking layer while maintaining
compatibility with existing applications.

---

# Design Constraints

The following constraints must always hold:

## Constraint 1

The EventLoop must never block.

---

## Constraint 2

A single request executes on one worker thread.

---

## Constraint 3

Filter, DispatcherServlet and Controller execute on the same thread.

---

## Constraint 4

ThreadLocal-based infrastructure must observe the same behavior as in a
traditional Servlet container.

---

# Out of Scope

This project does not currently address:

- Fully reactive business logic
- Reactive database drivers
- Reactive transaction management
- Hybrid reactive/blocking execution

These topics require a fundamentally different application architecture and are
outside the current project goals.

---

# Summary

The threading model deliberately separates transport processing from business
execution.

RSocket provides an efficient networking layer.

Spring MVC continues to provide a familiar blocking execution model.

A single thread switch between these two worlds preserves compatibility,
reduces complexity and minimizes migration cost.
