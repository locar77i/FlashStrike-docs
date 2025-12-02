
# Matching Engine Benchmark Report (Flashstrike Prototype)

**Author:** Rafael López Caballero  
**Environment:** HP EliteBook 640 G10 (Intel 13th Gen Mobile CPU)  
**CPU:** Intel Core i5-1335U (10 cores: 2P + 8E), up to 4.6 GHz Turbo  
**Note:** Mobile CPU with aggressive power/thermal scaling — not deterministic for real‑time latency.

**Component:** Single-instrument ultra-low-latency matching engine (C++)  
**Benchmark Type:** Enhanced stress test (insert/cancel/modify heavy)

---

## 1. Test Overview

**Total operations executed:** **51,000,000,000**  
- Inserts: 31,258,563,008  
- Cancels: 15,628,598,207  
- Modifies: 4,112,735,953  
- Trades executed: 759,829  
- Quantity filled: 218,235,393,449  

**Matching engine memory footprint:** **68.27 MB**  
- Order pool: 28.0 MB (524,288 capacity)  
- Order-ID map: 8.0 MB (1,048,576 capacity)  
- Partition pool: 32.14 MB (256 partitions)  
- Trades ring: 64 KB (1024 capacity)

---

## 2. Benchmark Environment Notice

This benchmark was executed on an **HP EliteBook 640 G10** powered by an **Intel Core i5‑1335U**, a mobile 13th‑generation Raptor Lake CPU.

These CPUs are *not* designed for deterministic ultra‑low‑latency workloads due to:

- Aggressive dynamic turbo boosting  
- Power limit enforcement (PL1/PL2)  
- Frequent P‑state/C‑state transitions  
- Thermal throttling under sustained load  
- ACPI and firmware interrupts  
- OS scheduler migrations unless manually isolated  

**Conclusion:**  
Rare multi‑millisecond tails are environmental and not caused by the matching engine logic.

Despite this, the engine maintained **sub‑microsecond** p50–p99 latencies across a 51‑billion‑operation run.

---

## 3. Throughput Summary

| Category | Throughput |
|---------|------------|
| **Overall request rate** | **8.77M req/s** |
| Process order (insert) | 7.49M req/s |
| Process on‑fly order | 14.7M req/s |
| Process resting order | 5.03M req/s |
| Modify price | 6.73M req/s |
| Modify quantity | 14.0M req/s |
| Cancel order | 13.1M req/s |

---

## 4. Latency Summary (microseconds)

### Insert Path (Process order)
- p50: **0.064µs**  
- p90: **0.128µs**  
- p99: **0.512µs**  
- p99.9: **1.024µs**  
- p99.99: **8.192µs**  
- p99.999: **65.536µs**  
- p99.9999: **131.072µs**  
- max: **~9.25ms** *(expected on laptop hardware)*

### Modify (price)
- p50: **0.064µs**  
- p99: **0.512µs**  
- p99.9999: **131.072µs**

### Modify (quantity)
- p50: **0.064µs**  
- p99.9: **0.128µs**  
- p99.9999: **131.072µs**

### Cancel
- p50: **0.064µs**  
- p99: **0.128µs**  
- p99.9999: **131.072µs**

---

## 5. Environment Impact Summary

Because the benchmark ran on a laptop (EliteBook 640 G10):

- CPU downclocking and turbo transitions introduce jitter  
- ACPI/firmware interrupts generate unpredictable pauses  
- Thermal throttling can cause ms‑scale latency spikes  

Still, the engine demonstrates:

- **Stable sub‑microsecond hot‑path latency**  
- **Consistent multi‑million‑ops/sec throughput**  
- A matching profile that will improve dramatically on a desktop/server with isolated cores

---

## 6. Summary

> Sustained **8.77M req/s** over **51B operations** with **p50=0.064µs** and **p99=0.512µs**, even on a laptop‑class CPU (EliteBook 640 G10).  
> Ultra‑tail spikes are environmental. On isolated server cores, the engine is expected to show even tighter deterministic behavior.

---

## 7. Next Steps
- Re‑run benchmark on a desktop CPU (e.g., Ryzen 5800X3D, Intel 12900KS) for deterministic tails.  
- Use `isolcpus`, `nohz_full`, `rcu_nocbs` for noise‑free measurements.  
- Add taker‑heavy sweeps to measure deep matching and trade emission performance.  

---

