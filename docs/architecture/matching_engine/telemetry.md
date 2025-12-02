# Telemetry Architecture — FlashStrike Matching Engine

The FlashStrike telemetry system is designed for **HFT‑grade ultra‑low‑overhead metrics**, with strict cache‑line alignment and zero dynamic allocation during hot paths.

This document explains all telemetry components:

- `Telemetry`
- `Init`
- `Manager`
- `PriceLevelStore`
- `LowLevel`
- Updaters for each subsystem

---

# 1. Design Goals

FlashStrike telemetry is built for:

### ✔ **Zero‑overhead in the matching hot path**
All metric objects are:
- pre‑allocated,
- 64‑byte cache‑line aligned,
- updated through lock‑free atomic ops.

### ✔ **Measurable but isolated**
Metrics record:
- latencies,
- successes/failures,
- probe lengths,
- hit rates,
- partition operations,
- modifications,
- cancellations,
- matching events.

Cold‑path collectors can export or snapshot metrics without impacting
matching throughput.

### ✔ **Per‑component isolation**
Each subsystem updates only its own counters:
- OrderPool → `LowLevelUpdater::on_allocate_order`
- OrderIdMap → `LowLevelUpdater::on_insert_ordid`
- PriceLevelStores → `PriceLevelStoreUpdater::*`
- Manager → `ManagerUpdater::*`
- Initialization → `InitUpdater::*`

This eliminates cross‑component coupling.

---

# 2. Telemetry Root Struct

```cpp
struct Telemetry {
    telemetry::Init init_metrics;
    telemetry::Manager manager_metrics;
    telemetry::PriceLevelStore pls_asks_metrics;
    telemetry::PriceLevelStore pls_bids_metrics;
    telemetry::LowLevel low_level_metrics;
};
```

This object is injected into:

- **Manager**
- **OrderBook**
- **PriceLevelStores**
- **OrderPool**
- **OrderIdMap**
- **PartitionPool**

And passed as references to their respective updaters.

---

# 3. `Init` — Construction‑Time Metrics

Records one‑shot metrics:

- Creation latency of:
  - matching engine
  - order book
  - partition pool
  - order pool
  - order id map
  - trades ring
- Total memory footprint
- Max capacities

Updated via:

```cpp
InitUpdater::on_create_order_book()
InitUpdater::on_create_partition_pool()
...
```

### Why?
These metrics let you verify:

- consistent construction times,
- memory stability,
- predictable footprint per instrument.

---

# 4. `Manager` — High‑Level Matching Metrics

Tracks **order‑flow processing**, including:

- `process` throughput and latency
- `modify_price`, `modify_qty`, `cancel`
- Counts for:
  - full fills
  - partial fills
  - not found
  - rejected
- Trade counts per match

### Updated via `ManagerUpdater`

Hot‑path functions invoke:

```cpp
on_process_on_fly_order()
on_process_resting_order()
on_modify_order_price()
on_cancel_order()
on_match_order()
```

This provides deep observability of the high‑level matching engine.

---

# 5. `PriceLevelStore` Metrics

Collected separately for:

- **asks**
- **bids**

Metrics include:

- Insert limit order
- Remove order
- Resize order
- Reprice order
- Recompute global best
- Recompute partition best

### Updated via `PriceLevelStoreUpdater`

```cpp
on_insert_order<SIDE>()
on_remove_order<SIDE>()
on_recompute_global_best<SIDE>()
```

This enables precise profiling of order‑book topology dynamics.

---

# 6. `LowLevel` — Core Data‑Structure Metrics

Used by:
- `OrderPool`
- `OrderIdMap`
- `PartitionPool`

Tracks:

### OrderPool
- Allocation latency
- Release latency
- Current pool occupancy

### OrderIdMap
- Insert/remove latency
- Linear probe lengths
- Map size

### PartitionPool
- Partition allocate / release latency
- Partition count

Updated via:

```cpp
on_allocate_order()
on_insert_ordid()
on_allocate_partition()
```

Gives visibility into potential performance degradation as the book fills.

---

# 7. Memory Layout Constraints

Every telemetry struct:

- aligned to **64 bytes**,
- size aligned to **64 bytes**,
- each metric field aligned to prevent false sharing.

Verified by static asserts:

```cpp
static_assert(sizeof(Manager) % 64 == 0);
static_assert(alignof(PriceLevelStore) == 64);
```

This makes counters safe to update concurrently from different hot paths.

---

# 8. Metric Collection & Export

Each telemetry struct implements:

```cpp
template <typename Collector>
void collect(const std::string& prefix, Collector& collector) const noexcept;
```

This enables:

✔ Prometheus-like exporters  
✔ JSON dumps  
✔ CLI snapshots  
✔ Real‑time UI dashboards  

Metrics include hierarchical labels:

```
system: matching_engine
stage: init
direction: input/output
event: modify_price
...
```

---

# 9. Dumping (Human‑Readable)

Each struct implements a `dump()` method to print:

- latencies
- percentiles
- counts
- memory usage
- histogram summaries

Example:

```
[Matching Engine Metrics] Snapshot:
 Request processing  : 12,300,000 req/s
 Process order       : count=89M, avg=220ns
 ...
```

---

# 10. Integration in the Matching Engine

Telemetry is wired through constructor injection:

```
Manager
 ├── OrderBook
 │    ├── PriceLevelStore (bids)
 │    ├── PriceLevelStore (asks)
 │    ├── OrderPool
 │    ├── OrderIdMap
 │    └── PartitionPool
 └── Trades Ring
```

Each subsystem receives references to the metrics it is responsible for.

---

# 11. Why This Telemetry System Is HFT‑Grade

### ✔ Zero heap allocation  
### ✔ Const‑time counters, no locks  
### ✔ Cache‑aware layout  
### ✔ High‑resolution timestamps  
### ✔ Fully modular  
### ✔ Minimal hot‑path code branches  
### ✔ Scales linearly with number of instruments  

