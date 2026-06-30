# Upload and Download

## Why This Document

File transfer is fundamentally different from ordinary request processing.

Traditional Spring MVC applications typically use Multipart requests for file
upload and HttpServletResponse for file download.

These mechanisms work well in a short-lived HTTP request model, but are not the
best fit for a long-lived bidirectional transport such as RSocket.

This document describes the architectural principles for handling large data
transfer in this project.

---

# Design Objective

The architecture should distinguish between:

- business requests
- data transfer

These two responsibilities have different performance characteristics and should
not share the same execution model.

---

# Control Plane and Data Plane

This project separates communication into two logical planes.

```
                    Client
                      │
        ┌─────────────┴─────────────┐
        │                           │
        ▼                           ▼
 Control Plane                 Data Plane
        │                           │
        ▼                           ▼
 Spring MVC                 RSocket Stream
```

---

# Control Plane

The Control Plane is responsible for business semantics.

Typical responsibilities include:

- authentication
- authorization
- validation
- metadata
- business workflow
- persistence of business information

Requests handled by the Control Plane are relatively small and benefit from
Spring MVC's mature programming model.

---

# Data Plane

The Data Plane is responsible for transferring bytes.

Typical responsibilities include:

- file upload
- file download
- large binary objects
- streaming
- chunk transmission

The Data Plane should avoid introducing unnecessary buffering or object copying.

---

# Upload Workflow

```
Client
    │
    │ Upload Metadata
    ▼
Spring MVC
    │
    │ Validate Request
    ▼
Business Service
    │
    │ Generate Upload Context
    ▼
Client Starts Stream
    │
    ▼
RSocket Stream
    │
    ▼
Storage
```

Business information and binary data follow different paths.

---

# Download Workflow

```
Client
    │
    │ Download Request
    ▼
Spring MVC
    │
    │ Permission Check
    ▼
Business Service
    │
    │ Resolve Resource
    ▼
RSocket Stream
    │
    ▼
Client
```

Spring MVC determines *whether* a download is allowed.

The Data Plane determines *how* the data is transferred.

---

# Why Not Multipart?

Multipart is tightly coupled to the HTTP request model.

It assumes:

- request boundaries
- HTTP headers
- multipart encoding
- Servlet parsing

A persistent transport such as RSocket naturally supports streaming and does
not require multipart encoding.

Therefore multipart should not become the primary transport mechanism for large
files in this project.

---

# Streaming Advantages

Streaming provides several advantages.

## Constant Memory Usage

Large files can be transferred without loading the entire content into memory.

---

## Backpressure

The sender can automatically adapt to the receiver's processing speed.

This avoids excessive buffering.

---

## Long-lived Connections

A file transfer no longer depends on the lifetime of an HTTP request.

---

## Better Throughput

Continuous streaming avoids repeated request establishment and protocol
overhead.

---

# Business Responsibilities

Spring MVC continues to own all business decisions.

Examples include:

- whether upload is permitted
- maximum allowed file size
- ownership validation
- storage policy
- audit logging

These concerns remain ordinary business logic.

---

# Transport Responsibilities

The transport layer owns:

- chunk scheduling
- stream lifecycle
- retry strategy
- transmission flow control

The transport layer should never contain business rules.

---

# Failure Handling

Business failures and transport failures are different categories.

Examples of business failures:

- permission denied
- validation failure
- missing resource

Examples of transport failures:

- network interruption
- connection reset
- timeout
- stream cancellation

The architecture should distinguish these failures clearly.

---

# Design Principles

## Principle 1

Business logic belongs to Spring MVC.

---

## Principle 2

Binary data belongs to the transport layer.

---

## Principle 3

Metadata and binary streams should be processed independently.

---

## Principle 4

Large file transfer should avoid unnecessary buffering.

---

## Principle 5

Business workflows should not depend on transport implementation details.

---

# Current Scope

The initial implementation focuses on:

- upload authorization
- download authorization
- streaming transfer

Advanced features such as resumable upload and checkpoint recovery will be
evaluated separately.

---

# Summary

This project separates business semantics from binary data transfer.

Spring MVC continues to execute business workflows.

RSocket provides an efficient streaming transport for large binary payloads.

This separation improves scalability while preserving the existing application
architecture.
