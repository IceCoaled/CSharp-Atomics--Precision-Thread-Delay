# High-Performance Atomics & Precision Timing Library

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

A lightweight, high-performance C# library providing atomic operations and microsecond-precision thread delay capabilities. Designed for applications requiring extreme precision timing and thread-safe operations without the overhead of locks.

## Features

### Atomic Operations
- Thread-safe primitive operations without locks
- Support for multiple types: `AtomicInt32`, `AtomicInt64`, `AtomicDouble`, `AtomicFloat`
- High-performance implementation passing extensive concurrency tests
- Compare-and-exchange functionality for lock-free algorithms

### High-Precision Thread Delay
- Microsecond-level precision timing (up to 1408× more precise than `Thread.Sleep`)
- Adaptive implementation balancing CPU usage and timing accuracy
- Extremely low jitter (0.49%-2.37%) even at high frequencies
- Support for nanosecond, microsecond, millisecond, and second-level delays

## Installation

```bash
# Via NuGet (if published)
dotnet add package HighPerformanceAtomics

# Or clone the repository
git clone https://github.com/yourusername/high-performance-atomics.git
```

## Usage

### Atomic Operations

```csharp
// Create atomic variables
using var atomicInt = new AtomicInt32(0);
using var atomicLong = new AtomicInt64(0);
using var atomicDouble = new AtomicDouble(0.0);

// Thread-safe increment
atomicInt.Increment(1);
atomicLong.Add(100);

// Read current value
int currentValue = atomicInt.Read();

// Compare and exchange (CAS)
long expected = 100;
long update = 200;
long previous = atomicLong.CompareExchange(update, expected);
bool successful = previous == expected;
```

### High-Precision Delays

```csharp
// Microsecond delay (1/1,000,000 second)
Watch.MicroSleep(100); // Sleep for 100 microseconds

// Millisecond delay with much higher precision than Thread.Sleep
Watch.MilliSleep(1.5); // Sleep for 1.5 milliseconds

// Other timing options
Watch.NanoSleep(10000); // Sleep for 10,000 nanoseconds
Watch.SecondsSleep(0.01); // Sleep for 0.01 seconds

// Example: Creating a precise timer loop
const double targetFrameTime = 16666.67; // ~60 FPS in microseconds
while (running)
{
    var frameStart = Stopwatch.GetTimestamp();
    
    // Do frame processing here...
    
    var elapsed = (Stopwatch.GetTimestamp() - frameStart) * 1000000.0 / Stopwatch.Frequency;
    if (elapsed < targetFrameTime)
    {
        Watch.MicroSleep(targetFrameTime - elapsed);
    }
}
```

## Performance Benchmarks

### Atomic Operations

```
=======================================
1. Basic Thread Safety Test
=======================================
Starting test with 16 threads, 10000 operations each
Atomic counter: 160000, Expected: 160000
Atomic counter: 160000, Expected: 160000
Atomic counter: 160000, Expected: 160000
Atomic counter: 160000, Expected: 160000
Control counter: 160000, Expected: 160000
Test result: PASSED

=======================================
2.A High Contention Test
=======================================
High contention test: 64 threads all updating the same variable
Final value: 6400000, Expected: 6400000
Time taken: 1707ms
Test result: PASSED

=======================================
2.B Compare-And-Exchange Test
=======================================
CAS Operations - Successful: 6386, Failed: 1614
Final value: 28640

=======================================
3.B CPU Bound Test
=======================================
Final value: 12800000, Expected: 12800000
Test result: PASSED

=======================================
5. Stress Test (10 seconds)
=======================================
Starting stress test for 10 seconds
Stress test complete: 393822441 operations in 11162ms
Final value: -3150607813, Expected sum of operations: -3150607813
Result: PASSED
```

### High-Precision Delay

```
=======================================
High Precision Thread Delay Tests
=======================================
Testing MicroSleep accuracy at various durations...

Target (µs)     Avg Actual (µs) Error (%)       Consistency (%)
------------------------------------------------------------
1.00            43.14           4214.00         143.14
10.00           10.08           0.80            0.74
50.00           62.74           25.48           0.24
100.00          110.98          10.98           3.74
500.00          510.66          2.13            0.53
1000.00         1008.30         0.83            0.73
5000.00         5000.22         0.00            0.00
10000.00        10000.40        0.00            0.00

Comparing precision between different sleep methods...

Average error from target of 1ms over 100 trials:
Thread.Sleep         14.080000      ms
Watch.MilliSleep     1.011201       ms
Watch.MicroSleep     0.009994       ms

Relative precision improvements:
MilliSleep is 13.92x more precise than Thread.Sleep
MicroSleep is 1408.85x more precise than Thread.Sleep

Testing high-frequency timing capability...

Testing frequency: 1000Hz (period: 1000.00 µs)
Average interval: 1005.25 µs (target: 1000.00 µs)
Standard deviation: 4.96 µs (jitter: 0.49%)
Min/Max: 1000.20/1020.10 µs, Range: 19.90 µs

Testing frequency: 10000Hz (period: 100.00 µs)
Average interval: 103.52 µs (target: 100.00 µs)
Standard deviation: 2.45 µs (jitter: 2.37%)
Min/Max: 100.00/115.70 µs, Range: 15.70 µs
```

## Implementation Details

### Atomic Operations
The atomic implementations use the `Interlocked` class internally but provide a more convenient API and support for additional types. Each operation ensures proper memory ordering and visibility across all CPU cores.

### High-Precision Thread Delay
The `Watch.MicroSleep` implementation uses an adaptive approach:
- For very short delays (<1.5ms): Uses a combination of `Thread.SpinWait` with optimized spin counts
- For longer delays: Uses `Thread.Yield()` to minimize CPU usage while maintaining precision
- Dynamically adjusts behavior based on system load and contention

## Use Cases

- **Game Development**: Frame timing, physics simulations, input handling
- **Financial Systems**: High-frequency trading, real-time market data processing
- **Audio/Video Processing**: Media synchronization, streaming
- **Scientific Applications**: Precise experimental timing, data acquisition
- **Parallel Computing**: Lock-free algorithms, concurrent data structures

## Requirements

- .NET 6.0+ or .NET Framework 4.7.2+
- Works with both 32-bit and 64-bit processes

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Inspired by Java's atomic classes and high-resolution timing mechanisms
- Special thanks to the .NET runtime team for the Interlocked API

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
