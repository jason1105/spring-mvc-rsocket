# Project Engineering Principles

## 1. Documentation First Principle

> Documentation is the primary artifact of this project.
> Code is only an implementation of documented decisions.

No feature is allowed to be implemented without:
- design explanation
- architecture reasoning
- risk analysis

## 2. ADR Requirement

All non-trivial decisions must be documented as ADR:

- Why this design exists
- What alternatives were considered
- What trade-offs were made

## 3. Connector SPI First-Class Design

The system must be designed around:

> Connector SPI as the core abstraction boundary

RSocket is only one implementation.

## 4. Compatibility Constraint

All Spring MVC features must remain unchanged:

- Controllers
- Filters
- Security
- Validation
- Exception handling

## 5. No Framework Forking Rule

We do NOT fork Spring MVC.
We do NOT modify DispatcherServlet behavior.

We only adapt input layer below it.

## 6. Evolution Model

Architecture evolves in this order:

Idea → ADR → Diagram → Implementation → Benchmark

