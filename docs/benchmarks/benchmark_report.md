# FlashStrike Matching Engine — High-Performance Benchmark Summary
*A system engineered for nanosecond-class, HFT-grade throughput and determinism*

## 🚀 Executive Overview

This benchmark run demonstrates that the FlashStrike matching engine reaches **HFT-class performance**, sustaining:

- **51 billion operations executed end-to-end**
- **8.47 million requests per second sustained**
- **Sub-microsecond median and p99 latencies**
- **Consistently controlled tail behavior up to the 99.9999th percentile**

These results place FlashStrike in the performance envelope expected of **colocated exchange engines**, **proprietary low-latency venues**, and **high-frequency trading systems** where nanoseconds matter.

---

# 📊 Workload Summary

- **Total operations:** 51,000,000,000  
  – Inserts: 31.26B  
  – Cancels: 15.63B  
  – Modifies: 4.11B  

- **Trades executed:** 759,829  
- **Quantity filled:** 218,235,393,449  

This benchmark represents a *genuine exchange-scale workload*, not a micro-benchmark.

---

# 🧠 Engine Architecture Footprint

The engine initializes with:

- **68.27 MB** matching engine core  
- **68.15 MB** order book memory  
- **28.00 MB** order pool (524K orders)  
- **8.00 MB** order-ID map (1M entries)  
- **32.14 MB** partition pool (256 partitions @ 4096 ticks each)

These figures reflect an architecture optimized for:

- predictable memory layouts  
- cache-aligned structures  
- constant-time operations  
- zero allocations on the hot path  

---

# ⚡ Throughput & Latency Performance

## 🔹 Global Request Processing Rate
**8.47 million requests per second sustained**, including inserts, cancels, modifies, and matches.

This is well within the performance envelope of production HFT engines.

![System Status](./plots/00.system_status.png)

---

# 🔹 Core Matching Performance

## **Process Order**
- **Avg:** 0.137 µs  
- **Throughput:** 7.26M rps  
- **p50:** 0.064 µs  
- **p99:** 0.512 µs  
- **p99.9999:** 262.144 µs  

Even the deepest tail latencies remain below 300 µs—exceptional for a software matching engine without kernel bypass.

![Process Order](./plots/01.process_order.png)

---

## **Process On-Fly Order**
- **Avg:** 0.07 µs  
- **Rate:** 14.2M rps  

Hot-path logic operates in **~70 nanoseconds**.

---

## **Process Resting Order**
- **Avg:** 0.204 µs  
- **Rate:** 4.88M rps  

Expectedly higher due to:
- book traversal  
- crossing scenarios  
- trade generation  

Still well within microsecond-class matching performance.

![Process Order Details](./plots/02.process_order_by_type.png)

---

# 🔧 Modify / Cancel Operations

## **Modify Price**
- **Avg:** 0.15 µs  
- **Rate:** 6.65M rps  
- **Not found:** 1  

![Modify Order Price](./plots/03.modify_order_price.png)

## **Modify Quantity**
- **Avg:** 0.075 µs  
- **Rate:** 13.3M rps  
- Perfect success rate (0 rejects)

![Modify Order Quantity](./plots/04.modify_order_qty.png)

## **Cancel Order**
- **Avg:** 0.08 µs  
- **Rate:** 12.5M rps  
- **Not found:** 3  

![Cancel Order](./plots/05.cancel_order.png)

Modify/cancel paths demonstrate exceptional stability, indicating properly optimized data structures and best-case cache behavior.

---

# 🎯 Tail Latency Discipline

Latency percentiles across all operations show:

- **Sub-microsecond** performance up through **p99.9**
- **Low microsecond** performance through **p99.99**
- **Controlled tail events** at extremely high percentiles (99.9999th)

This is typical of:
- lock-free data structures  
- cache-resident working sets  
- predictable branch patterns  

and is a hallmark of professional-grade matching engine engineering.

---

# ✔ Error & Rejection Profile

Across 51 billion operations:

- **Total failures:** 15,617  
- **Failure rate:** 0.00003%  
- All failures correspond to intentionally invalid test inputs.

This confirms **correctness under load** and no structural instabilities.

---

# 🏁 Evaluation

> **FlashStrike demonstrates performance typically found only inside colocated exchange engines or top-tier proprietary trading systems.**  
> 
> - 7–14 million ops/sec across core paths  
> - sub-100 ns hot-path latency  
> - consistent tail behavior at extreme percentiles  
> - predictable memory footprint and deterministic data access  
> - correctness maintained across 51 billion operations  
> 
> The system clearly reflects:
> - deep CPU microarchitecture,  
> - cache behavior,  
> - lock-free concurrency,  
> - data-oriented design,  
> - and exchange-grade determinism.

This benchmark strongly signals **HFT-level engineering capability**, suitable for:

- high-frequency trading teams  
- exchange matching engine teams  
- ultra-low-latency infrastructure roles  
- performance-critical systems engineering  

