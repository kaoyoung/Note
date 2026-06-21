# Workfolw

```text
Boot VM → Take golden snapshot
              ↓
  [For each experiment trial]
  Clear cache → Restore N VMs simultaneously
                     ↓
         Measure how long until first HTTP response
         (P50 / P95 / P99 latency)
              ↓
  Compare: Unscheduled vs FIFO vs Round-Robin scheduling
```
key term:
- **MicroVM**: a very lightweight virtual machine, boots in milliseconds
- **Snapshot**: a frozen saved state of a VM, like hibernate on a laptop
- **Working set**: the memory pages a VM actually needs to run — prefetching loads these ahead of time
- **P99 latency**: the worst-case response time experienced by the slowest 1% of requests — what you're trying to improve
- **Tail latency**: same idea — the "long tail" of slow responses under load
- **NVMe bandwidth**: how fast your SSD can read data; when 16 VMs all read at once, they compete for this
# File layout
```text
~/fc-work/
└── scheduler/
    ├── io_coordinator.py    ← the daemon (the scheduler)
    ├── worker.sh            ← per-VM restore script (asks daemon for permission)
    ├── run_experiment.sh    ← launches N workers + measures latency
    └── results/
```

# How to get the VM IP 
- No tap devices exist
- No network scripts exist
- Networking was never configured in this project
>**Use API-based timing (what I just showed)** Measure time from `snapshot/load` to `Resumed` response. No networking needed, and it's actually the most accurate measure of restore latency for your project. This is the pragmatic choice.
# Alternative direction
1. Deadline scheduling
2. Adjust I/O chunk size as the main variable
3. Change where in Firecracker's restore pipeline you intervene
4. Prioritize by working set size, not arrival order
5. Optionally add networking later (not required for your project)


# 每組 20 trials, N 拉到 8 
for N in 2 4 8; do for policy in none fifo rr; do for trial in {1..20}; do ./run_experiment.sh $N $policy $trial done done done