## [1.0.0] - 2026-06-05

### Initial Release — Migrated from TRnK Toolkit

Timer system extracted from TRnK Toolkit into its own standalone package.

- `Countdown` — counts down from a duration to zero; supports looping, OnUpdate, OnComplete, OnUpdateWhen, AddTime/ReduceTime.
- `Stopwatch` — counts up until cancelled or a stop predicate fires; supports OnUpdate, OnComplete, OnUpdateWhen, SetStopWhen.
- `TimerToken` — cancel-only handle for fire-and-forget patterns.
- `TimerHandle` — internal versioned slot reference.
- `TimerConfig.SetCapacity(n)` — pre-allocate the slot array to avoid runtime growth.
- `MonoBehaviour.Delay(delay, action)` and `.Repeat(interval, action)` extension helpers.
- `TimerTrackerWindow` — editor window (Tools → TRnK → Timer Tracker) showing live countdown and stopwatch state.
- PlayerLoop-injected tick driver — no MonoBehaviour Update, no coroutines.
- ECS-style hot/cold slot split for cache-friendly per-frame traversal.
