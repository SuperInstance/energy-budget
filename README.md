# Energy Budget — Energy Budget Calculation and Tracking Utilities

`energy-budget` is a Rust crate for computing, tracking, and optimizing energy consumption in computing systems. It provides utilities for measuring per-operation energy costs, modeling power consumption over time, and making energy-aware scheduling decisions.

## Why It Matters

Energy is the dominant cost in modern computing:

- **Data centers** consume ~1% of global electricity (IEA, 2024), and AI training runs can cost millions in power alone.
- **Edge devices** (IoT sensors, mobile) are battery-constrained — every millijoule matters.
- **Sustainability regulations** (EU CSRD, SEC climate disclosures) require energy accounting.
- **ML inference** tradeoffs: a model that's 2% less accurate but 10x more energy-efficient is often the better production choice.

`energy-budget` provides the primitives to make these tradeoffs programmable rather than ad-hoc: measure energy per operation, compare alternatives, and schedule work within power envelopes.

## How It Works

### Energy Model

The energy consumed by an operation is modeled as:

$$E_{\text{total}} = E_{\text{static}} + E_{\text{dynamic}}$$

Where:
- **E_static** = P_idle × Δt (baseline power consumption during the operation)
- **E_dynamic** = P_peak × Δt × u (additional energy from active computation, modulated by utilization u ∈ [0,1])

For a sequence of operations, total energy is:

$$E_{\text{total}} = \sum_{i=1}^{n} (P_{\text{idle}} + P_{\text{active},i} \times u_i) \times \Delta t_i$$

### Power Metrics

| Metric | Formula | Description |
|---|---|---|
| Energy per operation | E_op = E_total / n | Average joules per operation |
| Energy efficiency | η = operations / E_total | Operations per joule |
| Power draw | P = E / Δt | Instantaneous power (watts) |
| Energy-delay product | EDP = E × Δt | Classic efficiency metric for VLSI |

### Scheduling Under Energy Constraints

Given a set of tasks T = {t₁, ..., tₙ} with energy costs e(tᵢ) and a global energy budget B:

**Decision problem:** Is there a schedule S ⊆ T such that Σ_{t ∈ S} e(t) ≤ B?

This is the **0/1 knapsack problem** (NP-complete in general), solvable in:
- **O(nB)** via dynamic programming (pseudo-polynomial)
- **O(n log n)** via greedy if all tasks have equal value density (fractional relaxation)

For real-time scheduling under both time and energy constraints, the problem becomes **NP-hard** (dual-constraint scheduling).

### Big-O Summary

| Operation | Complexity |
|---|---|
| Single operation energy measurement | O(1) |
| Budget check (remaining vs. required) | O(1) |
| Task set feasibility (greedy) | O(n log n) |
| Optimal task selection (DP knapsack) | O(n × B) |
| Continuous power integration | O(t) where t = samples |

## Quick Start

```toml
[dependencies]
energy-budget = "0.1"
```

```rust
// Currently a stub crate. Planned API:
fn main() {
    println!("Hello, world!");
}

// Planned usage:
// let budget = EnergyBudget::new(Duration::from_secs(3600))
//     .max_watts(65.0)
//     .build();
//
// let task = Task::new("inference")
//     .estimated_joules(1200.0)
//     .deadline(Duration::from_secs(60));
//
// if budget.can_run(&task) {
//     budget.execute(&task).await?;
// }
```

## API

### Planned Types

| Type | Description |
|---|---|
| `EnergyBudget` | Time-bounded energy envelope with power ceiling |
| `Task` | Unit of work with estimated joules and deadline |
| `PowerMeasurement` | A single (timestamp, watts) sample |
| `EnergyReport` | Aggregated consumption summary |

### Planned Methods

| Method | Description |
|---|---|
| `EnergyBudget::new(duration)` | Create a budget for a time window |
| `budget.max_watts(p)` | Set peak power ceiling |
| `budget.can_run(&task)` | Check if task fits remaining budget |
| `budget.execute(&task)` | Run task, debit energy |
| `budget.remaining()` | Remaining joules in budget window |
| `budget.report()` | Generate EnergyReport |

## Architecture Notes

`energy-budget` implements **γ + η = C**:

- **γ (gamma)**: The energy model — the mathematical specification of how power, time, and utilization relate to total energy. This defines the physics of the budget.
- **η (eta)**: The Rust implementation — RAPL readings, `Duration` arithmetic, async task execution with energy bookkeeping. This is the measurement and enforcement machinery.
- **C (Configuration)**: **Energy-aware scheduling** — the operational property that emerges when the energy model (γ) correctly informs the scheduler (η). When aligned, tasks are admitted or deferred based on real energy constraints, preventing thermal throttling and battery depletion.

### Planned Integration Points

| Interface | Method | Use Case |
|---|---|---|
| `#[measure_energy]` proc macro | Automatic wrapping | Per-function energy attribution |
| `EnergyBudget` as async sentinel | RAII guard | Auto-debit on task completion |
| Prometheus exporter | `/metrics` endpoint | Dashboard energy monitoring |
| Intel RAPL / AMD APMC | MSR reads | Hardware-level joule counting |

## References

- **Murgai, R., et al. (2015).** "Power Analysis and Optimization for VLSI Circuits." *IEEE Trans. on VLSI Systems.* — Energy-delay product foundations.
- **Pedram, M. (1996).** "Power Minimization in IC Design: Principles and Applications." *ACM Trans. on Design Automation of Electronic Systems*, 1(1). — Dynamic and leakage power modeling.
- **Ranganathan, P., & Chang, C. (2021).** "Revisiting Energy Efficiency in the AI Era." *IEEE Micro.* — Modern energy metrics for ML workloads.
- **IEA. (2024).** *Electricity 2024: Analysis and Forecast.* International Energy Agency. — Data center energy consumption statistics.
- **Keller, J., et al. (1998).** "Maximizing Throughput While Staying within Power and Thermal Constraints." *Proc. ISCA Workshop on Complexity-Effective Design.* — Dual-constraint scheduling.
- **Cormen, T. H., et al. (2022).** *Introduction to Algorithms*, 4th ed., Ch. 35 (Approximation Algorithms, knapsack). MIT Press.
- **Patterson, D. A., & Hennessy, J. L. (2020).** *Computer Organization and Design*, 6th ed., Ch. 5 (Power and Energy). Morgan Kaufmann.

## License

MIT
