---
layout: default
title: FlashStrike Documentation
nav_order: 1
---

# FlashStrike — High-Performance Matching Engine Documentation

Welcome to the official documentation for **FlashStrike**, an ultra-low-latency, memory-deterministic matching engine designed for HFT and crypto exchanges.

## 📚 Architecture Overview

The following sections describe each internal component of the engine:

[Matching Engine Overview](docs/architecture/matching_engine_overview.md)

### Core Components

- [Manager](docs/architecture/matching_engine/manager.md)
- [Order Book](docs/architecture/matching_engine/order_book.md)
- [Price Level Store](docs/architecture/matching_engine/price_level_store.md)
- [Partition System](docs/architecture/matching_engine/partitions.md)
- [Order Pool](docs/architecture/matching_engine/order_pool.md)
- [Order ID Map](docs/architecture/matching_engine/order_id_map.md)
- [Telemetry System](docs/architecture/matching_engine/telemetry.md)

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

## ⚡ Benchmark & Performance Report

FlashStrike includes a public benchmark report demonstrating the engine’s
ultra-low-latency behavior under exchange-scale workloads.

👉 Read the full report:
[High-Performance Benchmark Report](docs/benchmarks/benchmark_report.md)

---

Happy reading!  
— *R. Lopez Caballero*


---

