# `NormalizedInstrument` Struct Documentation

This document provides an in-depth description of the `flashstrike::matching_engine::conf::NormalizedInstrument` structure — the fully integer‑scaled, cache‑aligned representation of a tradable asset pair used internally by the Flashstrike Matching Engine.

---

## Overview

`NormalizedInstrument` is the **normalized, engine‑ready** counterpart of `Instrument`.  
Where `Instrument` stores human‑readable floating‑point values (e.g. `0.01 USD` tick size),  
**`NormalizedInstrument` stores the same values in deterministic integer form**, scaled for:

- integer‑only arithmetic  
- CPU cache efficiency  
- predictable performance  
- avoidance of floating‑point rounding issues  

The matching engine and order book *never* use floating‑point values.  
This struct is the authoritative domain for all price, quantity, and notional constraints.

---

## Memory & Layout Guarantees

The struct enforces:

- `sizeof(NormalizedInstrument) <= 64`
- `alignas(64)` ensures the struct fits **entirely inside one CPU cache line**
- `std::is_trivially_copyable_v`
- `std::is_standard_layout_v`

This guarantees:

- zero‑overhead copying  
- no hidden pointers or vtable  
- no padding surprises  
- optimal locality for hot‑path matching operations  

---

## Field Summary

Although all fields are private, they are exposed through constexpr accessors.

### Price Domain

| Field | Type | Description |
|-------|------|-------------|
| `price_tick_size_` | `Price` | Integer size of one price tick |
| `price_min_scaled_` | `Price` | Minimum scaled price (usually equal to 1 tick) |
| `price_max_scaled_` | `Price` | Maximum scaled price |

### Quantity Domain

| Field | Type | Description |
|-------|------|-------------|
| `qty_tick_size_` | `Quantity` | Integer quantity tick |
| `qty_min_scaled_` | `Quantity` | Minimum allowed scaled quantity |
| `qty_max_scaled_` | `Quantity` | Maximum allowed scaled quantity |

### Notional Domain

| Field | Type | Description |
|-------|------|-------------|
| `min_notional_` | `Notional` | Minimum scaled notional order value |

---

## Accessor Methods

All accessors are `constexpr` and `noexcept`, allowing compile‑time evaluation where possible.

### Price Domain Accessors

```cpp
constexpr Price price_tick_size() const noexcept;
constexpr Price price_min_scaled() const noexcept;
constexpr Price price_max_scaled() const noexcept;
```

### Quantity Domain Accessors

```cpp
constexpr Quantity qty_tick_size() const noexcept;
constexpr Quantity qty_min_scaled() const noexcept;
constexpr Quantity qty_max_scaled() const noexcept;
```

### Notional Accessor

```cpp
constexpr Notional min_notional() const noexcept;
```

---

## Helper Methods

### `update_price_upper_limit(std::uint64_t num_ticks)`

Updates `price_max_scaled_` to ensure the domain aligns with a tick‑partitioned layout:

```cpp
inline constexpr void update_price_upper_limit(std::uint64_t num_ticks) noexcept {
    price_max_scaled_ = static_cast<Price>(num_ticks * price_tick_size_);
}
```

This is often used when constructing a `PartitionPlan`.

---

## Relationship to `Instrument`

`NormalizedInstrument` is a friend of `Instrument` and can only be constructed/modified by it.

Normalization process performed by:

```cpp
Instrument::normalize(std::uint64_t num_ticks)
```

This computes:

- integer tick sizes  
- price/quantity min/max in scaled form  
- min notional in the combined integer domain  

and writes those values into a `NormalizedInstrument` instance.

---

## Example Usage

```cpp
Instrument inst = ...;
auto norm = inst.normalize(1'000'000);

// Access normalized integer values
Price pstep = norm.price_tick_size();
Quantity qmin = norm.qty_min_scaled();
Notional nmin = norm.min_notional();
```

---

## Structural Safety Checks

The following static assertions ensure correctness and performance:

```cpp
static_assert(sizeof(NormalizedInstrument) <= 64);
static_assert(alignof(NormalizedInstrument) == 64);
static_assert(std::is_trivially_copyable_v<NormalizedInstrument>);
static_assert(std::is_standard_layout_v<NormalizedInstrument>);
```

These assertions guarantee the struct is safe for:

- memcpy  
- lock‑free queues  
- tight matching loops  
- cache‑optimized memory layouts  

---

## Summary

`NormalizedInstrument` provides the **engine‑ready, integer‑scaled**, cache‑aligned representation of a market.  
It is the foundation for any deterministic, high‑performance, floating‑point‑free matching engine logic.

