# FlashStrike Matching Engine — Public Architecture Documentation

This repository contains the **public-facing architecture documentation** for the
FlashStrike ultra-low-latency matching engine.

⚠️ **Note:**  
This repository contains **documentation only**.  
The production source code remains private.

---

## 📚 Documentation Index

- [Matching Engine Overview](docs/architecture/matching_engine_overview.md)

### Core Components
- [Manager](docs/architecture/matching_engine/manager.md)
- [OrderBook](docs/architecture/matching_engine/order_book.md)
- [OrderPool](docs/architecture/matching_engine/order_pool.md)
- [OrderIdMap](docs/architecture/matching_engine/order_id_map.md)
- [PriceLevelStore](docs/architecture/matching_engine/price_level_store.md)
- [Partitions & PartitionPool](docs/architecture/matching_engine/partitions.md)
- [Telemetry System](docs/architecture/matching_engine/telemetry.md)

---

## 🎯 Purpose

This documentation is intended to:

- Showcase the design of a production-grade matching engine  
- Highlight low-latency techniques and data-structure decisions  
- Demonstrate systems engineering expertise for prospective employers  
- Provide a structured, browsable architecture reference  

The underlying engine implementation is **closed-source**.

---

## ⚡ Benchmark & Performance Report

FlashStrike includes a public benchmark report demonstrating the engine’s
ultra-low-latency behavior under exchange-scale workloads.

📈 51 billion operations executed end-to-end
⚡ 8.47 million requests/second sustained
🎯 Sub-microsecond median & p99 latency
🏆 HFT-grade deterministic tail behavior

👉 Read the full report:
[High-Performance Benchmark Report](docs/benchmarks/benchmark_report.md)

![System Status](docs/benchmarks/plots/00.system_status.png)

### This report is intended to showcase:
- real-world performance characteristics,
- data structure efficiency,
- tail-latency control,
- and the engine’s suitability for HFT-class workloads.

---

## 🌍 Live Documentation Site

If this repository is published via **GitHub Pages**, the rendered site will be available at:

https://locar77i.github.io/FlashStrike-docs/

