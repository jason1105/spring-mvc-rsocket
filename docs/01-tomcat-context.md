# Tomcat & Servlet Container Context

## 1. What is a Servlet Container?

A Servlet Container is a runtime system that:

> Converts network requests into Java method execution chains.

It is responsible for:
- parsing HTTP
- creating HttpServletRequest/Response
- managing FilterChain lifecycle
- invoking Servlet (DispatcherServlet in Spring)

## 2. Tomcat Architecture Layers

### 2.1 Connector Layer (Coyote)
Responsible for:
- socket handling
- HTTP parsing
- request object creation

### 2.2 Container Layer (Catalina)
- Engine
- Host
- Context (web application isolation)

### 2.3 Servlet Pipeline
- FilterChain
- Servlet execution

### 2.4 Application Layer
- Spring MVC
- Controller
- Service

## 3. Key Insight

> The most important role of Tomcat is NOT HTTP handling.
> It is lifecycle orchestration of requests.

## 4. Mapping to This Project

This project replaces ONLY:

### Replaced:
- Coyote Connector (HTTP input layer)

### Reused:
- FilterChain
- DispatcherServlet
- Entire Spring MVC stack

## 5. Implication

We are effectively building:

> A virtual Servlet Container input layer.

