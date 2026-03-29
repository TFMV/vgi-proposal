## [PROPOSAL] WebSocket Transport for Apache Arrow Flight

### Summary

This proposal suggests adding an optional WebSocket transport for Apache Arrow Flight to enable browser-native and lightweight client access while preserving existing gRPC semantics.

---

### Motivation

Apache Arrow Flight provides a high-performance RPC framework over gRPC for columnar data transfer. However, direct browser support remains limited due to lack of native gRPC support in browser environments.

Today, browser-based usage typically requires intermediary layers (e.g., Node.js proxies or gRPC-Web gateways), which introduce additional operational complexity and latency.

As interactive, browser-first data applications become more common, this gap is increasingly limiting.

---

### Problem

Flight’s reliance on gRPC makes it difficult to use directly from browser environments, resulting in:

* Additional proxy infrastructure
* Increased system complexity
* Reduced end-to-end streaming efficiency
* Fragmented client implementations

There is interest in a simpler path for browser and lightweight clients to consume Arrow Flight streams directly.

---

### Proposal

Introduce an optional **WebSocket transport binding for Arrow Flight** that:

* Preserves existing Flight semantics (DoGet, DoPut, DoExchange, Ticket, FlightDescriptor, etc.)
* Uses WebSocket as a transport layer for streaming Arrow IPC messages
* Supports binary streaming of RecordBatch, Schema, and related IPC messages
* Coexists with existing gRPC transport without modification to core semantics

This is not a replacement for gRPC, but an additional transport option.

---

### Design Goals

1. **Protocol Compatibility**

   * Maintain existing Flight message types and semantics without modification.

2. **Browser Compatibility**

   * Enable direct use of standard WebSocket APIs in browsers.

3. **Streaming Support**

   * Support bidirectional streaming of Arrow IPC messages over a persistent connection.

4. **Minimal Overhead**

   * Prefer a direct mapping of Arrow IPC messages to WebSocket binary frames where possible.

5. **Optional Transport**

   * Servers may expose both gRPC and WebSocket endpoints concurrently.

---

### Sketch of Approach

* Client establishes a WebSocket connection to a Flight endpoint (e.g., `wss://host/flight`).
* Initial handshake carries Flight metadata equivalent to existing gRPC metadata where applicable.
* Flight operations (e.g., DoGet) are expressed as message sequences over the WebSocket connection.
* Arrow IPC messages are transmitted as binary frames.
* Each stream maintains ordering and backpressure semantics consistent with Flight behavior.

This is an initial sketch intended for discussion and refinement.

---

### Implementation Reference

A working implementation exists in **Porter**, a Flight SQL server built on DuckDB, which supports:

* gRPC-based Flight SQL transport
* WebSocket-based streaming transport
* Shared execution and planning layer across both transports

This demonstrates feasibility of mapping Flight semantics onto WebSocket transport without changes to core execution logic.

---

### Open Questions

Feedback is requested on the following:

* Message framing: 1:1 IPC-to-frame mapping vs. lightweight batching
* Authentication: reuse of existing Flight auth mechanisms vs. browser-native approaches
* Backpressure: representation over WebSocket transport
* Error mapping: translation between Flight status codes and WebSocket semantics
* URI scheme: formalization of `ws://` / `wss://` in Flight Location definitions
* Standardization scope: formal specification vs. recommended extension pattern

---

### Related Work

This proposal is intended to be complementary to existing Flight and gRPC-Web efforts, not a replacement.

---

### Motivation Summary

This work aims to reduce friction for browser-native and lightweight clients consuming Arrow Flight streams, without changing the core Flight execution model.

---

### Discussion

If there is interest from the community, I would be happy to:

* Contribute a formal specification draft
* Iterate on the Porter reference implementation
* Provide performance comparisons between gRPC and WebSocket transports

---

### Minimal Client Model

At minimum, a browser client should be able to:

* Connect to a Flight endpoint
* Issue a Ticket or FlightDescriptor
* Receive a stream of Arrow RecordBatches
* Consume results incrementally

WebSocket transport is proposed as a way to simplify this path while preserving Flight semantics.
