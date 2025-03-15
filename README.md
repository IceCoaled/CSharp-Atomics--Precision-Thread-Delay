# High-Performance Atomics & Precision Timing Library

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

A lightweight, high-performance C# library providing atomic operations and microsecond-precision thread delay capabilities. Designed for applications requiring extreme precision timing and thread-safe operations without the overhead of locks.

## Features

### Atomic Operations
- Thread-safe primitive operations without locks
- Support for multiple types: `AtomicInt32`, `AtomicInt64`, `AtomicDouble`, `AtomicFloat`, `AtomicThreadSignal`
- High-performance implementation passing extensive concurrency tests
- Compare-and-exchange functionality for lock-free algorithms

### High-Precision Thread Delay
- Microsecond-level precision timing (up to 1673x more precise than `Thread.Sleep`)
- Adaptive implementation balancing CPU usage and timing accuracy
- Extremely low jitter (0.49%-2.37%) even at high frequencies
- Support for nanosecond, microsecond, millisecond, and second-level delays


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
Control counter: 160000, Expected: 160000
Test result: PASSED

=======================================
2.A High Contention Test
=======================================
High contention test: 64 threads all updating the same variable
Final value: 6400000, Expected: 6400000
Time taken: 1258ms
Test result: PASSED

=======================================
2.B Compare-And-Exchange Test
=======================================
CAS Operations - Successful: 5537, Failed: 2463
Final value: 24897

=======================================
2.C Complex Operations Test
=======================================
After complex operations, final value: 1810522506872750077

=======================================
3.A Delay Injection Test
=======================================
Running with 32 threads, 1000 operations each
Starting threads...
Atomic result: 32000, Expected: 32000
Non-atomic result: 973, Expected: 32000
Atomic Test Result: PASSED
Non-atomic Test Result: FAILED
Race condition statistics:
- Lost operations: 31,015
- Percentage lost: 96.92%

=======================================
3.B CPU Bound Test
=======================================
Final value: 12800000, Expected: 12800000
Test result: PASSED

=======================================
5. Stress Test (10 seconds)
=======================================
Starting stress test for 10 seconds
Stress test complete: 387215854 operations in 10890ms
Final value: -3098135902, Expected sum of operations: -3098135902
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
1.00            35.20           3420.00         143.22
10.00           10.10           1.00            1.08
50.00           54.78           9.56            1.79
100.00          100.18          0.18            0.07
500.00          507.64          1.53            0.51
1000.00         1006.34         0.63            0.18
5000.00         5000.28         0.01            0.00
10000.00        10000.26        0.00            0.00

Testing CPU usage patterns during high precision delays...

Measuring operations per second with different delay types:
Delay Type      Ops per Second
------------------------------
No delay        38,650,366
Thread.Sleep(0) 2,570,410
Thread.Sleep(1) 97
MicroSleep(1)   971,219
MicroSleep(50)  19,134
MicroSleep(1000) 994

Comparing precision between different sleep methods...

Average error from target of 1ms over 100 trials:
Thread.Sleep         9.020000       ms
Watch.MilliSleep     1.007837       ms
Watch.MicroSleep     0.005390       ms

Relative precision improvements:
MilliSleep is 8.95x more precise than Thread.Sleep
MicroSleep is 1673.47x more precise than Thread.Sleep

Testing different sleep variant methods...

Target (ms)     NanoSleep       MicroSleep      MilliSleep      SecondsSleep
---------------------------------------------------------------------------
1.00            1.05            1.00            1.00            1.03
10.00           10.00           10.00           10.00           10.00
100.00          100.00          100.00          100.00          100.00

Testing high-frequency timing capability...

Testing frequency: 1000Hz (period: 1000.00 µs)
Average interval: 1006.64 µs (target: 1000.00 µs)
Standard deviation: 4.99 µs (jitter: 0.50%)
Min/Max: 1000.00/1028.70 µs, Range: 28.70 µs

Testing frequency: 2000Hz (period: 500.00 µs)
Average interval: 506.36 µs (target: 500.00 µs)
Standard deviation: 2.07 µs (jitter: 0.41%)
Min/Max: 500.20/513.70 µs, Range: 13.50 µs

Testing frequency: 5000Hz (period: 200.00 µs)
Average interval: 201.55 µs (target: 200.00 µs)
Standard deviation: 1.69 µs (jitter: 0.84%)
Min/Max: 200.10/211.60 µs, Range: 11.50 µs

Testing frequency: 10000Hz (period: 100.00 µs)
Average interval: 105.73 µs (target: 100.00 µs)
Standard deviation: 2.38 µs (jitter: 2.25%)
Min/Max: 100.00/110.60 µs, Range: 10.60 µs
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
- .NET 8.0 or later
- This is due to the use of primary constructors, if you need it on earlier version
i made adjustments for the `Thread _Lock` to allow the use of the old `Object _Lock`.
The interfaces used for arithmetic / bitwise operations require NET 7.0 as well.
## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments
- Special thanks to the .NET runtime team for the Interlocked API

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
