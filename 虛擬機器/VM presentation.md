# Chapter 8
1. Why we need contiguous memory allocation?
Contiguous allocation is chosen for **implementation simplicity** of pointer chasing, not because it guarantees physical contiguity. The actual cache coverage argument relies on page offset preservation (within pages) and statistical coverage (across pages), which holds regardless of how many VM translation layers exist.






---
# Slide 10

# Slide 11

# Slide 12






---
# Slide 10 — Step 2: Side-Channel Attacks

Now we move to the second step of the attack — extracting information from the victim once co-residence is achieved.

The authors use a technique called Prime+Trigger+Probe to measure cache activity on the shared physical machine. The idea is straightforward. The attacker and victim share the same CPU cache, and by carefully measuring how the cache behaves, the attacker can infer what the victim is doing.

Here is how it works. First, the attacker allocates a buffer B and reads through it completely, which loads B into the CPU cache. This is the Prime step. Now the attacker owns the cache.

Next, the attacker enters a busy-loop, continuously checking the CPU's cycle counter and waiting for a big jump. A sudden jump means the Xen scheduler has preempted the attacker's VM and handed the CPU to another VM — hopefully the victim. This is the Trigger step.

When the attacker gets the CPU back, it immediately re-reads buffer B and measures how long it takes. If the latency is high, it means the victim was actively computing during the trigger window and evicted the attacker's data from the cache, forcing a slow fetch from main memory. If the latency is low, the victim was idle and the cache is still intact. This is the Probe step.

One thing worth noting — the authors read buffer B in pseudorandom order using pointer-chasing. This prevents the CPU prefetcher from predicting the access pattern and pre-loading data ahead of time, which would hide the true cache state and make the measurement unreliable.

Using this single measurement technique, the authors demonstrate three attacks, shown in the table. The first is cache-based co-residence detection, which confirms whether two instances are on the same physical machine without any network packets. The second is traffic rate estimation, where the attacker infers how much web traffic a co-resident site is receiving — achieving a clear correlation across zero to two hundred requests per minute. The third is keystroke timing, where the attacker recovers inter-keystroke intervals from an SSH session, with a five percent miss rate and thirteen millisecond resolution. We will go through each of these in the following slides.

---

# Slide 11 — Side-Channel Attack 1: Cache-based Co-residence Detection

The first application is using cache measurements to confirm co-residence purely through hardware, with no network packets involved at all.

Why does this matter? Recall that in the placement section, the authors used network-based co-residence checks — specifically, comparing Dom0 IP addresses via traceroute. Those checks work well, but a cloud provider could easily defeat them by hiding Dom0 from traceroutes or randomizing internal IP assignments. If network-based checks are blocked, the attacker needs an alternative. This cache-based approach provides exactly that — it works regardless of how the network is configured.

The method relies on being able to induce load on the target. If the target is running a public web server, the attacker can send HTTP requests to it from an external machine, causing the target's CPU to do work. Meanwhile, the attacker VM, sitting on the same physical machine, is continuously taking Prime+Trigger+Probe measurements. If the measurements spike when HTTP requests are being sent and drop when they stop, the target is co-resident. If the measurements stay flat regardless of load, the target is somewhere else.

The experiment uses three pairs of m1.small instances. Trials 1 and 2 are co-resident pairs on two different physical machines. Trial 3 is a non-co-resident pair. Each pair takes 100 measurements with HTTP load and 100 without.

Looking at the graphs, Trials 1 and 2 show a clear separation between the two sets of measurements — the red points, taken during HTTP load, are consistently higher than the green points. Trial 3 shows no separation at all — the two sets are indistinguishable, exactly as expected for non-co-resident instances.

What's particularly impressive here is that these measurements were taken on live EC2 instances, with unknown background activity from other VMs on the same machine. You can even see a few unexplained spikes in Trial 2 that the authors attribute to a third co-resident instance doing its own work. Despite this noise, the signal is clear and reliable.

---

# Slide 12 — Side-Channel Attack 2: Traffic Rate Estimation 

The second attack takes this a step further. Rather than just detecting whether a co-resident VM is under load, the attacker tries to estimate how much traffic it is receiving — essentially spying on a competitor's web traffic in real time.

Think about why this is sensitive. Traffic volume is not protected by encryption. It reveals things like when a product was launched, whether a marketing campaign is working, what hours a service is most active, or whether a business is growing. **A competitor with this information has a significant advantage, and the victim has no way of knowing the leak is happening**.

The experiment is clean and well-controlled. Two co-resident m1.small instances are used. The target runs Apache and serves a three megabyte text file — **the large file size amplifies the CPU load per request, making the signal easier to detect**. The attacker takes 1,000 cache measurements at each of four different traffic rates: zero, fifty, one hundred, and two hundred HTTP requests per minute. **The whole measurement window is about 90 seconds per rate**. The experiment is repeated three times to check reproducibility.

The results tell a compelling story. **Across all three trials, the mean CPU cycle count rises consistently as traffic rate increases**. At zero requests per minute, the mean sits around 300,000 CPU cycles. At 200 requests per minute, it reaches nearly 750,000. The three trials track each other closely, confirming the results are stable and not just noise. **The reason is straightforward — higher traffic on the target VM causes more cache evictions during the Trigger phase, which means when the attacker re-reads the buffer in the Probe phase, more of it has been evicted from cache, driving up the measured CPU cycle count**.

The authors are careful to note that this is preliminary — they are estimating traffic rate rather than identifying specific pages or individual users. **But the principle is established: an attacker can use cache measurements to infer continuous, real-time information about a co-resident workload, and that information may be highly valuable.**

---

# Slide 13 — Side-Channel Attack 3: Keystroke Timing

The third attack is the most direct in terms of impact on individual users. The goal is to detect individual keystrokes typed into a co-resident VM's terminal using cache measurements alone.

The key idea is that on an otherwise idle machine, every keystroke causes a brief spike in cache activity, because the victim VM's operating system processes the input. **The attacker uses Prime+Trigger+Probe to detect these spikes in real time**. Specifically, the authors report a keystroke when the probe latency falls between 3.1 and 9 microseconds — **below that threshold the machine is idle, and above it the activity is likely unrelated system noise rather than a keystroke**. The authors also noted that there is a clear difference between different types of key events — for example, typing a shell command produces a different pattern than pressing Enter to execute it.

The attacker does not directly learn which keys were pressed. However, the inter-keystroke timing alone is enough to feed into the password recovery method proposed by Song et al. in 2001. **That work showed that timing information from SSH sessions is sufficient to significantly narrow down password candidates** — and the 13 millisecond resolution achieved here meets that requirement.

Overall the results show a 5% missed keystroke rate and only 0.3 false triggers per second, which is quite reliable for a hardware side channel in a noisy shared environment.

There is one important caveat. This experiment was not run directly on EC2. It was conducted on a local pinned Xen testbed with the same CPU and hypervisor configuration as EC2 m1.small instances. **The reason is that this attack requires the attacker and victim to be scheduled on the same physical core, and EC2 migrates virtual CPUs across the machine's four cores unpredictably.** That condition is only satisfied about 25% of the time. The authors acknowledge this as a limitation, but point out that a **patient attacker could simply wait for the right scheduling alignment to occur** — and given that the vCPU assignment changes frequently, the attacker would not have to wait long.

---

# Slide 14 — Defenses (~2 min)

So what can be done about these attacks?

The paper proposes three categories of defense. The first is obfuscating the internal IP structure — randomizing IP assignments, hiding Dom0 from traceroutes, and isolating accounts via VLANs. This makes cloud cartography harder and defeats the network-based co-residence checks we saw in slide 8. However, as we just showed in slide 11, the attacker can fall back to hardware-based co-residence detection using cache timing alone. So this only slows the attacker down, it does not stop them.

The second is cache blinding — wiping the cache between VM timeslices, inserting random delays, or blurring the cycle counter. The fundamental problem is that the cache is just one of many shared resources. The memory bus, branch predictor, instruction cache, and disk all also leak information. It is extremely difficult to guarantee that every possible channel has been anticipated and closed, and new channels are discovered regularly.

The third option, and the only one the authors consider fully reliable, is dedicated instances — letting users pay to occupy a physical machine exclusively with their own VMs. This eliminates co-residence entirely. The authors calculate that for a large user, the overhead amounts to at most one additional physical machine, making the relative cost increase small. This is the only foolproof solution.

---

# Slide 15 — Conclusion (~1 min)

To summarize, this paper makes three key contributions.

First, it shows that cloud infrastructure is mappable. An attacker with only standard customer access can build a detailed picture of EC2's internal structure and use it to target specific victims.

Second, it demonstrates that co-residence leaks real information. Cache side channels can reveal whether two VMs are on the same machine, how much traffic a website receives, and even the timing of individual keystrokes — all without any direct access to the victim.

Third, and most importantly, software isolation through the hypervisor is not enough. Physical isolation through dedicated single-tenant deployment is the only complete fix, and the authors argue this option should be explicitly offered to customers with strong privacy requirements.

---
# Appendix A
So this slide explains how the cache covert channel actually works reliably in EC2's noisy environment.

The basic problem is straightforward. EC2 machines run many VMs at the same time, and all of them are sharing the same CPU cache. So when you try to measure cache activity, you're picking up noise from every other VM on the machine — not just your target. The simplest approach, where the sender idles to signal a zero and frantically accesses memory to signal a one, completely falls apart in this kind of environment because you can't tell apart the target's signal from all the background noise.

So the authors came up with a smarter approach using what they call Odd and Even cache set encoding. The idea is to split all the cache sets into two groups based on the memory address. Even sets correspond to addresses where the address mod 2d equals zero, and Odd sets correspond to addresses where it equals d.

Here's the key part. To transmit a zero, the sender reads Even addresses, which evicts the Even cache sets. To transmit a one, the sender reads Odd addresses, which evicts the Odd sets. The receiver then measures the read time for Even sets minus the read time for Odd sets. If the difference is positive, Even was slower, meaning Even was evicted, so the sender transmitted a zero. If the difference is negative, Odd was slower, so the sender transmitted a one.

Now why does this help with noise? Because noise from unrelated VMs hits both Even and Odd sets roughly equally. When you subtract one from the other, that common noise cancels out, and what you're left with is only the sender's asymmetric signal — the deliberate imbalance between the two groups.

The result is a covert channel achieving around 0.2 bits per second on live EC2 instances, which is a significant improvement over the basic approach and reliable enough to actually use in a real attack scenario.

