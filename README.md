# TRnK Timer

PlayerLoop-driven countdown and stopwatch timers for Unity. No coroutines, no MonoBehaviour Update — timers are managed in a flat ECS-style slot array ticked directly from Unity's PlayerLoop.

## Installation

### Via Git URL

```
https://github.com/trnkdev/unity-trnk-timer.git
```

### Manual Installation

1. Download the package
2. Import into your Unity project

---

## Timers

```csharp
using TRnK.Timer;
```

Two timer types, both lightweight `readonly struct` handles:

- **`Countdown`** — counts down from a duration to zero; supports looping.
- **`Stopwatch`** — counts up until cancelled or a stop predicate fires.

```csharp
// Countdown — fires OnComplete once per iteration
var countdown = Countdown.Create(this, 10f)
    .SetLoop(3)
    .OnUpdateWhen(() => !isPaused)
    .OnUpdate(remaining => UpdateUI(remaining))
    .OnComplete(() => OnPeriodElapsed());

countdown.Start();

// Stopwatch — fires OnComplete when stopWhen becomes true
var stopwatch = Stopwatch.Create(this)
    .SetStopWhen(() => gameIsOver)
    .OnUpdate(elapsed => UpdateElapsedDisplay(elapsed))
    .OnComplete(() => OnGameTimeStopped());

stopwatch.Start();
```

## Timer Control

```csharp
// State queries
bool alive   = countdown.IsAlive;
bool running = countdown.IsRunning;
bool paused  = countdown.IsPaused;

// Countdown values
float remaining  = countdown.RemainingTime;
float total      = countdown.TotalTime;
int   iteration  = countdown.CurrentLoopIteration;

// Countdown time adjustments — NaN/negative are ignored; ReduceTime clamps to zero
countdown.AddTime(5f);
countdown.ReduceTime(2f);

// Stopwatch values
float elapsed = stopwatch.ElapsedTime;

// Control — Cancel is silent (no callbacks fire)
countdown.Pause();
countdown.Resume();
countdown.Cancel();

// Cancel-only handle — safe to pass to code that shouldn't control the timer
TimerToken token = countdown.AsTimerToken();
token.IsAlive;   // check if still running
token.Cancel();  // same effect as countdown.Cancel()
```

## Callback Semantics

| Callback | Countdown | Stopwatch |
|---|---|---|
| `OnUpdate(float)` | Every tick — receives remaining seconds | Every tick — receives elapsed seconds |
| `OnComplete()` | Every iteration boundary (1× one-shot, N× finite loop, ∞ infinite loop) | Once when `SetStopWhen` predicate becomes true |
| `OnUpdateWhen(predicate)` | Gates ticking; timer pauses internally while false | Same |

`Cancel()` and owner-MonoBehaviour destruction are silent — no callbacks fire.

## Invoke Helpers

`MonoBehaviour` extension methods for fire-and-forget patterns:

```csharp
// Invoke once after a delay
TimerToken token = this.Delay(2f, () => SpawnEnemy());
TimerToken unscaled = this.Delay(2f, () => SpawnEnemy(), useUnscaledTime: true);
token.Cancel(); // cancel before it fires

// Repeat every interval
TimerToken ticker = this.Repeat(1f, () => TickRegeneration());
ticker.Cancel(); // stop the loop
```

Notes:
- `Delay(delay <= 0)` fires immediately and returns `default`.
- `Repeat(interval <= 0)` throws `ArgumentException`.

## Non-Capturing Callbacks

All callbacks have generic overloads to avoid closure allocations:

```csharp
// Instead of capturing 'this' in a lambda:
countdown.OnUpdate(this, static (self, remaining) => self.UpdateUI(remaining));
countdown.OnComplete(this, static self => self.OnComplete());
countdown.OnUpdateWhen(this, static self => !self.IsPaused);
```

## Configuration

Pre-allocate the slot array at startup to prevent runtime growth:

```csharp
// Call once at game startup (e.g. in a bootstrap MonoBehaviour Awake)
TimerConfig.SetCapacity(128);
```

Default capacity is 32 — the array doubles automatically when full, with a warning.

## Timer Tracker (Editor)

Open via **Tools → TRnK → Timer Tracker** during Play Mode to inspect all live countdowns and stopwatches: slot index, version, owner GameObject/component, remaining/elapsed time, loop progress bar, and status.

---

## Requirements

- Unity 6 or later
- [TRnK Toolkit](https://github.com/trnkdev/unity-trnk-toolkit) (`com.trnkdev.unitytoolkit`)

## License

See [LICENSE.md](LICENSE.md) for license info.
