# Proposal: VGI Execution Layer over Arrow-Native QUIC Transport

## Overview

This proposal outlines a clean separation of concerns between:

* A **general-purpose Arrow-native transport layer over QUIC**
* A **DuckDB-backed execution layer (VGI)** built on top of that transport

The goal is to align on a minimal, composable architecture that can serve both as:

1. A **reference implementation** for distributed DuckDB execution (VGI)
2. A **foundation for standardizing Arrow-native streaming over QUIC** across the broader ecosystem

---

## Key Idea

Separate **data transport** from **query execution**:

* Transport should be **database-agnostic** and reusable
* Execution should be **pluggable** and optimized for DuckDB (initially)

---

## Architecture

### Layer 1: Arrow-Native Transport over QUIC

A minimal transport abstraction for streaming Arrow data:

**Core responsibilities:**

* Bidirectional streaming over QUIC
* Arrow IPC message framing
* Schema + RecordBatch transmission
* Flow control and backpressure
* Stream lifecycle management

**Conceptual interface:**

```
OpenStream() -> Stream

Stream:
  SendSchema(schema)
  SendRecordBatch(batch)
  Recv() -> (schema | batch)
  Close()
```

**Properties:**

* Transport-agnostic interface (QUIC as first implementation)
* Zero-copy friendly
* Multiplexed streams
* Backpressure-aware

---

### Layer 2: VGI (DuckDB Execution Layer)

A thin execution interface built on DuckDB and Arrow.

**Core abstraction:**

```
Execute(query string) -> ArrowStream
```

**Responsibilities:**

* Execute queries using DuckDB
* Emit results as Arrow RecordBatches
* Map execution to transport streams
* Support unary and streaming queries

**Key idea:**

VGI acts as a modern equivalent of CGI:

* Input: query / function call
* Execution: DuckDB (with extensions)
* Output: Arrow stream

---

## Design Principles

### 1. Arrow-Native End-to-End

* No row-based translation
* Columnar data flows through the entire system

### 2. Streaming First

* Results are streamed incrementally
* Avoid full materialization when possible

### 3. Composability

* Nodes can call other nodes
* Streams can be pipelined across services

### 4. Minimal Surface Area

* Keep interfaces small and explicit
* Avoid premature abstraction

---

## MVP Scope

### Transport

* QUIC-based implementation
* Single stream per query
* Arrow IPC framing

### VGI Node

* Embedded DuckDB
* Single endpoint:

```
Execute(query) -> stream
```

* Supports:

  * Unary queries
  * Streaming result sets

---

┌──────────────────────────────┐
│ Client / Query               │
└─────────────┬────────────────┘
              │
              ▼
┌──────────────────────────────┐
│ VGI API                      │
│ Execute / Producer / etc     │
└─────────────┬────────────────┘
              │ Arrow streams
              ▼
┌──────────────────────────────┐
│ Transport Layer              │
│ streams + framing + QoS      │
└─────────────┬────────────────┘
              ▼
┌──────────────────────────────┐
│ Arrow IPC contract           │
│ schema + batches only        │
└─────────────┬────────────────┘
              ▼
┌──────────────────────────────┐
│ DuckDB runtime               │
└──────────────────────────────┘

## Future Extensions

### Distributed Execution

* Query delegation across nodes
* Pipeline execution (node-to-node streaming)

### Extension Execution

* Remote invocation of DuckDB extensions
* Treat extensions as network-accessible primitives

### Transport Generalization

* Alternative transports (TCP, Unix sockets)
* Standardization of Arrow streaming protocol

---

## Strategic Positioning

* **VGI** becomes the reference model for DuckDB-based distributed execution
* **Arrow-over-QUIC** becomes a reusable transport primitive for the broader ecosystem

This allows both layers to evolve independently while remaining tightly integrated.

---

## Summary

This proposal defines:

* A **clean execution boundary** for DuckDB (VGI)
* A **reusable transport layer** for Arrow streaming over QUIC

Together, they enable:

* High-performance, streaming query execution
* Composable distributed data pipelines
* A potential standard for Arrow-native transport systems

---

## Open Questions

* Stream framing details for Arrow IPC over QUIC
* Backpressure semantics across distributed pipelines
* Error propagation and cancellation model
* Query planning vs delegation boundaries

---

## Next Steps

1. Implement minimal Arrow-over-QUIC transport
2. Build VGI prototype on top of transport
3. Validate streaming performance under load
4. Iterate on protocol and execution model
