# FlashStrike OrderBook — Architecture Overview

## 1. Purpose
The FlashStrike `OrderBook` is a **preallocated, deterministic, O(1)** data structure that stores all resting orders and enables extremely fast insert, modify, cancel, and match operations.  
It is designed for **ultra-low-latency matching**, strict price–time priority, and predictable tail behavior.

This document summarizes the architecture and key design choices of the OrderBook.

---

## 2. High-Level Design

The OrderBook is built on top of three core, fixed-size memory structures:

```
OrderPool      - Stores all orders (contiguous, preallocated)
OrderIdMap     - Maps OrderID → OrderPool index (O(1) lookup)
PartitionPool  - Preallocated storage for all price-partition buckets
```

Each side of the book (**BID** and **ASK**) owns a dedicated `PriceLevelStore`:

```
PriceLevelStore<BID>
PriceLevelStore<ASK>
```

These stores manage:
- price levels grouped by partition  
- FIFO queues for strict time priority  
- O(1) best-price discovery  
- per-level aggregate quantity tracking  

This two-sided split avoids unpredictable branching and guarantees cache-friendly access patterns.

---

## 3. Memory Architecture

### ✔ Fully Preallocated
All OrderBook memory is allocated at startup:
- no heap allocation in the hot path
- no vector growth
- no node-based structures
- no fragmentation

This ensures **stable, deterministic latency** even under peak load.

### ✔ Index-Based Navigation
Orders are addressed by numeric indices rather than pointers:
- prevents pointer chasing  
- enables compact packing  
- ensures safe, zero-cost relocation between price levels  

### ✔ Partitioned Price Space
Prices are mapped deterministically:

```
price → partition → price level
```

The PartitionPlan is computed once per instrument and enforces:
- bounded memory use  
- constant-time index computation  
- tight CPU cache locality  

---

## 4. Core Operations

### 4.1 Insert (O(1))
```
order_idx = order_pool.allocate()
order_idmap.insert(orderid → index)
price_level_store.insert_order(index, order)
```

No branching beyond SIDE selection, no dynamic allocation.

### 4.2 Reprice (O(1))
- Retrieve order index from ID map  
- Move order between price levels  
- Adjust level aggregates  
- No reallocation or structural changes  

Repricing is strictly side-specific to avoid unpredictable branches.

### 4.3 Resize (O(1))
- Directly update quantity  
- Update level totals  
- No movement between levels  

### 4.4 Remove (O(1))
```
price_level_store.remove_order(index)
order_idmap.remove(orderid)
order_pool.release(index)
```

Complete teardown in constant time, fully inlined.

---

## 5. PriceLevelStore Internals

Each `PriceLevelStore` maintains:

- A sorted array of partitions  
- Within each partition: a compact array of price levels  
- Within each price level:  
  - a FIFO queue of orders  
  - best-order pointer  
  - aggregated quantity  

### Guarantees:
- O(1) best-price lookup  
- O(1) insert/remove at price level  
- O(1) head-of-level access for matching  
- Stable price–time priority  

This is the same design applied by modern high-performance exchange engines.

---

## 6. Engine Properties

### ✔ Deterministic
No locks, no heap allocation, no syscalls.  
Matching thread is isolated and single-threaded.

### ✔ Predictable Tail Latency
Operations avoid:
- variable-length lists  
- dynamic memory  
- hash tables  
- branches on hot path  

### ✔ Designed for Parallelization
Multiple OrderBooks can run:
- per instrument  
- per partition  
- per CPU core  

Scaling is horizontal and trivial.

### ✔ Exchange-Grade Semantics
- strict price–time priority  
- real-time modified order re-ranking  
- complete ID-based access  
- no state ambiguity  

---

## 7. Complexity Summary

| Operation       | Complexity | Notes |
|-----------------|------------|-------|
| Insert          | O(1) | pool + idmap + level insert |
| Remove          | O(1) | level remove + pool release |
| Reprice         | O(1) | cross-level move |
| Resize          | O(1) | pure data mutation |
| Lookup by ID    | O(1) | direct index map |
| Best price      | O(1) | pointer maintained by store |

This is the foundation of the engine’s ultra-low latency.

---

## 8. Why This Architecture Matters
The FlashStrike OrderBook mirrors the architectural principles found in:
- NASDAQ INET  
- Kraken’s X-Engine  
- Coinbase’s matching engine  
- Jump Trading & Jane Street proprietary LOBs  
- Modern FPGA-based matchers  

It is:
- **simple**  
- **ultra-fast**  
- **deterministic**  
- **production-ready**  

Exactly what a high-performance matching engine needs.

---

## 9. Summary
FlashStrike’s OrderBook is designed for the exact requirements of a professional, low-latency exchange:

- preallocated memory  
- O(1) operations  
- price–time priority  
- bid/ask isolation  
- deterministic performance  

Combined with the `Manager` and the lock-free trade event pipeline, it forms the backbone of a modern, scalable matching engine.
