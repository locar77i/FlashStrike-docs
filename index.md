---
layout: default
title: FlashStrike Documentation
nav_order: 1
---

# FlashStrike — High-Performance Matching Engine Documentation

Welcome to the official documentation for **FlashStrike**, an ultra-low-latency, memory-deterministic matching engine designed for HFT and crypto exchanges.

## 📚 Architecture Overview

The following sections describe each internal component of the engine:

- [Matching Engine Overview](docs/architecture/matching_engine_overview.md)
- [Order Book](docs/architecture/order_book.md)
- [Order Pool](docs/architecture/order_pool.md)
- [Order ID Map](docs/architecture/order_id_map.md)
- [Partition System](docs/architecture/partitions.md)
- [Price Level Store](docs/architecture/price_level_store.md)
- [Manager](docs/architecture/manager.md)
- [Telemetry System](docs/architecture/telemetry.md)

---

## 🔎 Purpose

This documentation is intended for:

- Recruiters evaluating system design skills  
- Trading firms assessing low-latency architectures  
- Contributors or reviewers analyzing data structures  
- Engineers interested in high-performance C++ design

---

## 💡 Project Summary

FlashStrike implements:

- A fully preallocated, intrusive order pool  
- Partitioned price-level indexing for O(1) lookup  
- Lock-free SPSC trade event ring  
- High-resolution telemetry with cache-line alignment  
- Deterministic latency and memory footprint  

---

## **⚙️ Benchmarks**

- [MatchingEngine{BTC/USD}](docs/benchmarks/benchmark_report.md)


---

Happy reading!  
— *R. Lopez Caballero*


---

