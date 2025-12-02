# `Instrument` Struct Documentation

This document describes the `flashstrike::matching_engine::conf::Instrument` structure, a compact representation of a tradable asset pair used inside the Flashstrike matching engine.

## Overview

`Instrument` models a market such as **BTC/USD**, providing:
- Human-readable tick/precision information
- Bounds for prices and quantities
- Minimum order and notional requirements
- A normalization method that converts floating-point inputs into fully integer-scaled values

It maps loosely to Kraken’s “Tradable Asset Pair” schema but includes only fields needed for the matching engine.

## Field Summary

### Symbol Identifiers
| Field | Type | Description |
|-------|------|-------------|
| `base_symbol[5]` | `char[]` | Base currency (e.g., "BTC") |
| `quote_symbol[5]` | `char[]` | Quote currency (e.g., "USD") |
| `name[10]` | `char[]` | Market symbol (e.g., "BTC/USD") |

### Tick & Precision Settings
| Field | Type | Description |
|-------|------|-------------|
| `price_tick_units` | `double` | Minimum price increment in quote units |
| `qty_tick_units` | `double` | Minimum quantity increment in base units |
| `price_decimals` | `uint8_t` | Decimal precision for price |
| `qty_decimals` | `uint8_t` | Decimal precision for quantity |

### Bounds
| Field | Type | Description |
|-------|------|-------------|
| `price_max_units` | `double` | Maximum allowed price |
| `qty_max_units` | `double` | Maximum allowed quantity |
| `min_qty_units` | `double` | Minimum base quantity |
| `min_notional_units` | `double` | Minimum notional (quote) value |

### Market Metadata
| Field | Type | Description |
|-------|------|-------------|
| `status[16]` | `char[]` | Market status (e.g., "online") |

## Normalization Method

```cpp
NormalizedInstrument normalize(std::uint64_t num_ticks) const noexcept;
```

This converts human-friendly values into deterministic, integer‑scaled internal units.

### Steps Performed
1. Price tick normalization  
2. Quantity tick normalization  
3. Notional normalization  
4. Price max recalculation based on partition ticks

## Utility Methods
- `get_symbol()` — returns `"BASE/QUOTE"`
- `debug_dump()` — prints all fields
- `to_string()` — returns formatted string

## Example

```cpp
Instrument btcusd{
    "BTC", "USD", "BTC/USD",
    0.01, 0.0001,
    2, 4,
    200000.0, 100.0,
    0.0001, 5.0,
    "online"
};
auto normalized = btcusd.normalize(1'000'000);
```
