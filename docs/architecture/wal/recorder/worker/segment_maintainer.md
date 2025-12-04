# WAL Segment Maintainer — Background Persistence, Compression & Retention Worker

## Overview
The **`SegmentMaintainer`** is the WAL subsystem’s background worker responsible for:
- Persisting completed WAL segments to durable storage  
- Compressing old segments using LZ4  
- Enforcing hot/cold retention policies  
- Deleting expired compressed segments  
- Running completely off the hot path  

It ensures the main matching engine and WAL writer maintain ultra-low-latency performance while long‑latency filesystem tasks run asynchronously in a dedicated background thread.

---

## 1. Purpose
High-frequency trading engines generate massive event streams requiring safe, sequential persistence. However, filesystem operations such as:

- `fdatasync`
- `close`
- `LZ4 compression`
- directory scanning
- file deletion

can all introduce **millisecond-level stalls**.

`SegmentMaintainer` eliminates this latency by moving all heavy persistence and retention work into a separate thread that runs independently of the matching engine and WAL writer.

---

## 2. Responsibilities

### **✔ Persist completed WAL segments**
Processes segments from:

```
segments_to_persist_ : spmc_task_ring<std::shared_ptr<SegmentWriter>>
```

For each segment:
- `close_segment(sync=true)` → ensures durability
- releases memory-mapped resources
- removes reference so it can be destroyed safely

---

### **✔ Compress old hot segments**
When notified through:

```
segments_to_freeze_ : spmc_task_ring<std::string>
```

The maintainer:
- flushes the file to ensure durability
- compresses using **LZ4 frame format**
- deletes the original `.wal`
- inserts compressed file into cold retention pipeline

---

### **✔ Delete oldest compressed segments**
When exceeding cold retention limits, it removes `.lz4` files through:

```
segments_to_free_ : spmc_task_ring<std::string>
```

---

### **✔ Maintain hot & cold retention**
- Hot = uncompressed `.wal`  
- Cold = compressed `.lz4`

Retention limits:
- `max_segments_`  
- `max_compressed_segments_`  

These values are clamped to system-wide limits defined in `wal/constants.hpp`.

---

## 3. Segment Lifecycle

```
┌─────────────┐
│  WAL Writer │
└──────┬──────┘
       │ pushes completed SegmentWriter
       ▼
┌────────────────────┐
│ SegmentMaintainer  │
└──────┬──────┬──────┘
       │      │
       │      ├────────────┐
       ▼      ▼            ▼
Persist   Compress       Delete
Hot WAL   Old WAL       Old .lz4
```

Lifecycle states:

1. **Hot segment** — recently written, uncompressed
2. **Cold segment** — compressed with LZ4
3. **Expired segment** — removed from disk

---

## 4. Threading Model

### **Dedicated background thread**
Started with:

```cpp
maintainer.start();
```

Stops gracefully with:

```cpp
maintainer.stop();
```

### **Safe interactions**
- The maintainer thread pops tasks from ring buffers  
- WAL writer pushes tasks into those buffers  
- Consumer interactions are thread-safe by design  

### **Exponential backoff**
To avoid busy spinning when idle:

```
sleep_time = min_sleep ... max_sleep
```

---

## 5. Persistence Loop Logic

Pseudo-code for `persistence_loop_()`:

```
while !stop:
    if segment_to_persist:
         close_segment()
         did_work = true

    if segment_to_freeze:
         compress()
         delete original
         did_work = true

    if segment_to_free:
         delete compressed file
         did_work = true

    if did_work:
         sleep = min_sleep
    else:
         sleep = min(sleep * 2, max_sleep)
```

Idle loop efficiency is achieved via exponential backoff.

---

## 6. Compression Details

The maintainer supports:

### **LZ4 Block Compression**
Fast but lacks metadata.  
Used for extremely compact storage.

### **LZ4 Frame Compression (recommended)**
- Provides container framing
- Checksums and proper block management
- Compatible with external LZ4 tools

Compression uses streaming 1 MiB chunks to avoid memory spikes.

---

## 7. Safety & Data Integrity

Before compression, the maintainer:

1. Ensures the WAL file is durable via `fsync`
2. Ensures chain checksums remain intact
3. Deletes the original `.wal` only after successful compression

During shutdown:
- Remaining segments in buffers are processed
- Ensures no partial segments remain on disk

---

## 8. Telemetry Integration

If `ENABLE_FS1_METRICS` is enabled, the maintainer records:

- Loop iteration latency
- Segment close latency
- Compression success/failure statistics
- Hot→cold transition metrics
- Cold retention deletion metrics

This is essential for diagnosing WAL I/O bottlenecks.

---

## 9. API Summary

```cpp
SegmentMaintainer(
    const std::string& dir,
    size_t max_segments,
    size_t max_compressed_segments,
    spmc_task_ring<std::shared_ptr<SegmentWriter>>& segments_to_persist,
    spmc_task_ring<std::string>& segments_to_freeze,
    spmc_task_ring<std::string>& segments_to_free,
    telemetry::worker::SegmentMaintainer& metrics
);
```

### Public API:

```cpp
void start();
void stop();
```

Everything else is handled internally.

---

## 10. Invariants

- Hot segments ≤ `max_segments_`
- Cold segments ≤ `max_compressed_segments_`
- WAL files must be non-empty
- All segments eventually persisted, compressed, or deleted
- Maintainer thread always drains all queues on shutdown

---

## 11. Related Documentation

- `wal_segment_writer.md` — hot-path segment writing  
- `wal_segment_preparer.md` — preallocates new segments  
- `wal_segment_header.md` — on-disk WAL header format  
- `wal_segment_block.md` — WAL block structure  
- `wal_segment_overview.md` — global architecture  

---

## Related components

[`recorder::Manager`](../manager.md)
[`recorder::SegmentWriter`](../segment_writer.md)
[`recorder::Meta`](../meta.md)
[`recorder::Telemetry`](../telemetry.md)

[`recorder::worker::MetaCoordinator`](./meta_coordinator.md)
[`recorder::worker::SegmentPreparer`](./segment_preparer.md)

---

👉 Back to [`WAL Recorder System — Overview`](../recorder_overview.md)
