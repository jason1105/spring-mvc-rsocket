# Spring MVC RSocket Connector Framework

## Vision

This project aims to decouple Spring MVC from HTTP by introducing a pluggable Connector SPI.

## Core Concept

Spring MVC is treated as a runtime engine:
- DispatcherServlet remains unchanged
- Transport layer is replaceable via Connector SPI

## First Implementation

RSocket is the first experimental Connector implementation.

## Architecture Principle

- MVC layer: unchanged
- Servlet semantics: preserved
- Connector layer: replaceable

## Status

Early architecture design phase (v0.1)

