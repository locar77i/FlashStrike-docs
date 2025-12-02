# Manager — FlashStrike Matching Engine Orchestrator  
High‑Level Component Architecture

## 1. Purpose

The **Manager** is the top-level orchestrator of the FlashStrike matching engine and responsible for:

- processing the market configuration ([see instrument.md](./conf/instrument.md))
- computing integer-scaled instrument parameters ([see normalized_instrument.md](./conf/normalized_instrument.md))
- generating the partitioning plan ([see partition_plan.md](./conf/partition_plan.md))
- initializing the order book and telemetry components
- preparing the system for processing orders at nanosecond precision

It is the *entry point* for all order operations and coordinates the behavior of the entire subsystem stack:

- Order validation  
- New order processing (MARKET / LIMIT)  
- Matching algorithm execution  
- Resting order insertion into the OrderBook  
- Price / quantity modifications  
- Cancellations  
- Trade event emission via a lock‑free SPSC ring  
- Metrics + telemetry collection  

It exposes a stable, deterministic interface suitable for HFT workloads and exchange integration.

---

## 2. Architectural Role

The Manager sits at the top of the pipeline:

```
┌─────────────────────────┐
│         Manager         │
│  (API / Orchestration)  │
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│       OrderBook         │
│ (bids/asks, pools, map) │
└───────────┬─────────────┘
            │
   ┌────────▼────────┐
   │ PriceLevelStore │
   └────────┬────────┘
            │
   ┌────────▼────────┐
   │  PartitionPool  │
   └────────┬────────┘
            │
   ┌────────▼────────┐
   │    OrderPool    │
   └─────────────────┘

+ Lock‑Free SPSC Trades Ring
+ Telemetry Subsystem
```

Manager does **not** maintain market structure internally—it delegates stateful work to the OrderBook while enforcing logic, sequencing, matching, and validation.

---

## 3. Core Responsibilities

### 3.1 Order Validation  
The Manager ensures that incoming orders satisfy instrument constraints:

- price range  
- quantity range  
- min notional  
- tick size checks (optional)  

Invalid orders are rejected immediately.

---

### 3.2 New Order Handling

For an incoming order (`BID` or `ASK`), Manager performs:

1. Validate order  
2. Execute matching loop (via templated `match_order_<SIDE>()`)  
3. If MARKET or fully filled → return result  
4. If LIMIT w/ remaining qty → insert into OrderBook via intrusive pools  
5. Emit trade events for each match  
6. Update metrics  

Templates ensure **zero runtime branching** between BID/ASK paths.

---

### 3.3 Matching Algorithm

Matching follows strict price‑time priority:

- Fetch best opposite‑side price level  
- Check crossing (via `PriceComparator<SIDE>::crosses()`)  
- Match incoming order vs oldest resting order  
- Update quantities  
- Remove fully filled resting orders  
- Emit trade events  
- Continue until no crossing remains

All operations are **O(1)** per matched order thanks to:

- intrusive order lists  
- price‑partition design  
- best‑price tracking  

---

### 3.4 Modify Orders  
Supports:

- **modify price**  
- **modify quantity**  

Price modifications may cause crossing → Manager triggers immediate matching.

Quantity modifications update the PriceLevelStore and maintain total quantity.

---

### 3.5 Cancel Orders  
A simple fast call:

```
book_.remove_order(orderid)
```

which performs:

- O(1) ID lookup (OrderIdMap)  
- O(1) removal from intrusive list  
- O(1) release of order slot  

---

### 3.6 Emit Trade Events

Trade events are published into a **lock‑free SPSC ring**:

- producer = Manager  
- consumer = external publisher thread  

The design ensures:

- no locks  
- minimal latency  
- bounded memory  
- deterministic behavior  

Busy‑spin + yield behavior handles buffer pressure smoothly.

---

### 3.7 Metrics & Telemetry

Manager integrates with the telemetry layer:

- order processing latency  
- match loop counts  
- partition usage  
- ring buffer usage  
- pool utilization  
- per‑million‑request diagnostic callback  

This enables real‑time health monitoring and performance profiling.

---

## 4. Detailed Internal Workflow

### 4.1 `process_order()`

Pseudo‑sequence:

```
validate order
on_request_()           // periodic metrics hook
match incoming order
if MARKET or full fill:
    return
else:
    insert resting order using OrderBook
```

The templated version avoids side‑based branching.

---

### 4.2 `match_order_<SIDE>()`

Algorithm steps:

```
best = best opposite price level
while best exists AND incoming crosses AND qty > 0:
    trade_qty = min(incoming.qty, resting.qty)
    update incoming / resting
    update price level qty
    emit trade event
    if resting empty → remove
    best = next best
```

Highly optimized to avoid:

- unnecessary heap use  
- unpredictable branching  
- pointer indirection  

---

### 4.3 `modify_order_price()`  

Steps:

1. Reprice the order via OrderBook  
2. If crossing occurs → call `match_resting_order_<SIDE>()`  
3. Update price level totals  
4. Cancel if fully executed  

---

### 4.4 `cancel_order()`  

Direct call into the pools → O(1).

---

## 5. Memory Model

Manager owns:

- `OrderBook`  
- SPSC trades ring  
- PartitionPlan + Instrument configs  
- Telemetry updaters  
- Sequence generator for trade events  

No dynamic memory allocation in hot path.  
All memory is preallocated at startup.

---

## 6. Determinism & Performance

Manager design ensures:

- **zero heap allocations**  
- **templated hot path**  
- **intrusive data structures**  
- **lock‑free communication**  
- **O(1) access patterns**  
- **low branch misprediction**  
- **tight CPU cache locality**

This is the same architecture strategy used in:

- equities exchanges  
- crypto exchanges (Kraken, Binance, Coinbase)  
- HFT firm internal engines  
- FPGAs / hardware‑accelerated books  

---

## 7. Summary

The FlashStrike Manager is:

- the **control center** of the matching engine  
- a **deterministic, low‑latency orchestration layer**  
- a **zero‑overhead template‑based dispatcher**  
- tightly integrated with:  
  - OrderBook  
  - PriceLevelStore  
  - PartitionPool  
  - OrderPool  
  - Intrusive FIFO lists  
  - Lock‑free SPSC trade ring  
  - Telemetry system  

Its architecture is engineered for **exchange‑grade throughput**, capable of millions of operations per second with predictable latency.

