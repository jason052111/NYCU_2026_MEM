# FIO Storage Performance Benchmarking

This project provides a comprehensive performance analysis of a high-performance NVMe SSD using the Flexible I/O Tester (`fio`). The benchmarking explores various I/O patterns and optimization techniques to understand the physical and logical limits of modern storage hardware.

## 💻 Hardware Specification
- **Model:** ADATA XPG GAMMIX S70 PRO
- **Performance Category:** High-End Consumer SSD
- **Test Environment:** Linux / WSL2

---

## 📊 Q0: Initial Benchmarking Result
Based on the `q0.fio` execution, the device demonstrates characteristics of a **Solid State Drive (SSD)**.

```text
read: IOPS=12.9k, BW=50.5MiB/s (52.9MB/s)(1024MiB/20283msec)
    lat (usec): min=41, max=15953, avg=76.91, stdev=80.67
```

**Observation:**
The 4K direct read achieved a bandwidth of **52.9 MB/s** with an average latency of only **76.91 µs**. This extremely low latency is only possible on flash-based storage, as a mechanical HDD would require several milliseconds due to physical head movement.

---

## 🔬 Experiment Analysis Summary

### Q1: Read vs. Random Read
- **Sequential Read:** ~60.2 MB/s
- **Random Read:** ~44.0 MB/s

**Analysis:** Even on SSDs, random reads are slower due to **Prefetch Failure** and **FTL Lookup Overhead**. Scattered LBA accesses force the controller to query the mapping table more frequently, increasing latency.

### Q2 & Q3: Write Dynamics
- **Finding:** No significant performance drop in Random or Backward writing.
- **Why?** SSDs use a **Log-Structured FTL**. The controller abstracts logical addresses and physically writes incoming data sequentially to the next available erased NAND pages, regardless of whether the logical order is forward, backward, or random.

### Q4: Buffered vs. Direct I/O
- **Buffered I/O:** Drastically faster (GB/s range) because it hits the **OS Page Cache (RAM)**.
- **Direct I/O:** Reflects true hardware speed (~61 MB/s) by bypassing the system cache.
- **Random Access:** Buffering is ineffective for random reads as prefetching cannot predict the next scattered address, leading to frequent "cache misses."

### Q5 & Q6: Physical Characteristics
- **Uniform Performance:** Unlike HDDs, SSDs show consistent bandwidth across all file offsets (no "inner" vs. "outer" track performance gap).
- **Block Size (1K vs 4K):** 4K is significantly faster because it reduces **Command Overhead** and aligns with the native 4KB page size of modern NAND flash and OS memory pages.

---

## 🚀 Reaching Peak Performance (Q7)

By optimizing the `fio` configuration, we achieved the hardware's near-theoretical limit:
- **Max Bandwidth:** **6,796 MB/s**

### Key Optimization Pillars:
1. **Large Block Size (1M):** Maximizes payload-to-overhead ratio by reducing the total number of I/O commands issued.
2. **Asynchronous I/O (libaio):** Allows the CPU to fire requests continuously without blocking or waiting for each one to complete.
3. **High Queue Depth (32):** Unlocks **Internal Parallelism**, engaging multiple NAND flash dies and memory channels simultaneously.

---

## 💡 SSD vs. HDD Comparison (Summary)

| Feature | SSD (Electronic) | HDD (Mechanical) |
| :--- | :--- | :--- |
| **Random Read** | Fast (Electronic FTL mapping) | Extremely Slow (Seek Time + Rotational Latency) |
| **Backward Write** | No Impact | Massive degradation (Spinning platter direction) |
| **Inner Tracks** | No Impact (Uniform access) | Lower Bandwidth (Smaller circumference) |
| **Parallelism** | High (Multi-channel NAND) | Low (Single Actuator arm) |

---

## 🛠 Usage
To reproduce these results, run the provided `.fio` job files:

```bash
# Run specific test
fio q7.fio > output_q7
```
