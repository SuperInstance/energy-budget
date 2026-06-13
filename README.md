# Energy Budget

**A utility library for calculating and tracking energy consumption budgets** — providing the framework for monitoring power usage, setting allocation limits, and computing energy efficiency metrics across compute resources.

## Why It Matters

Energy management is becoming critical in modern computing. Data centers consume ~1% of global electricity. The EPA estimates that server power management can reduce consumption by 20-80%. At every scale — from IoT devices with battery constraints to hyperscale datacenters with PUE targets — tracking and budgeting energy is essential.

This library provides the scaffolding for an energy budgeting system: modeling power consumption, setting allocation limits per resource, and tracking usage against those limits. It establishes the patterns used by production systems like Kubernetes power management, AWS Compute Optimizer, and embedded RTOS power governors.

**Key concepts:**
- **Energy budget** — A total energy allocation (in joules or watt-hours) for a period
- **Power tracking** — Current instantaneous consumption (in watts)
- **Efficiency metrics** — Computation per joule (ops/J, FLOPS/W)
- **Budget enforcement** — Throttling or shutting down when budget is exhausted

## How It Works

The library is currently a foundational scaffold. The intended architecture follows the standard pattern for energy monitoring:

**Budget model:** Each resource (container, process, device) receives an energy budget — a total energy allocation for a time window. As the resource consumes power, the budget is depleted. When the budget reaches zero, the system can throttle CPU frequency, reduce batch sizes, or pause non-essential work.

**Integration points:** In a container runtime, the energy budget would be enforced via cgroup CPU limits (which translate to throttling), CPU frequency scaling (DVFS), or workload scheduling. In embedded systems, it interfaces with hardware power management units (PMUs) and battery fuel gauges.

## Quick Start

```rust
fn main() {
    // Energy budget framework entry point
    // Future: initialize power meters, set budget allocations,
    // and register energy consumption callbacks
    println!("energy-budget starting...");
}
```

## API

Currently in scaffolding phase. Planned API surface:

- `EnergyBudget::new(total_joules: f64) -> Self` — Allocate an energy budget
- `consume(&mut self, joules: f64) -> BudgetStatus` — Record energy usage
- `remaining() -> f64` — Query remaining budget
- `efficiency() -> f64` — Computation per joule

## Architecture Notes

This library provides energy management for SuperInstance's infrastructure monitoring layer. It integrates with the container runtime for per-container power tracking and with the metrics forwarder for reporting energy efficiency to observability platforms.

See the full architecture: [ARCHITECTURE.md](https://github.com/SuperInstance/SuperInstance/blob/main/ARCHITECTURE.md)

## License

MIT
