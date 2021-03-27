# Elastic Compute Cloud (EC2)

## Intro

- resizable compute capacity, scale in minutes up and down as requirements change
- Pricing models:
  - on demand: pay per hour/second, no commitment
  - reserved: capacity reservation, much cheaper (1-3 years)
  - spot: excess capacity. Dropped prices - movable. Can be terminated.
  - dedicated: physical servers, for server bound licenses
- on demand:
  - low cost, flexible without up front or long term commitments
  - short term, spiky, unpredictable workloads that cannot be interuppted
  - spikes
- reserved:
  - steady state, predictable usage
  - can make upfront payments to reduce their overall cost
  - types:
    - standard: 75% off, greater discount if you buy for longer
    - convertable: Allows you to change CPU, RAM.
    - scheduled: time windows. fraction of week/day/month
- spot:
  - flexible start and end times
  - can be terminated
  - only feasible for very low compute prices. Urgent computing needs
- dedicated:
  - regulatory: no multi-tenant
  - licensing that doesn't support mult-tenent or cloud
  - can be purchased on demand (hourly)
  - reservation givres 70% off
- Instance types:
  - F1: Field Prgrammable I3: High Speed Storage, G3: Graphics Intensive, T3: Low cost/general, D2: Dense Storage, R5: Memory Optimised, M5: General, C5: Compute, P3: Graphics/General, X1: Memory, Z1D: High compute and memory, A1: ARM, U-6tb1: Bare metal
  - F: FPGA, I: IOPS, G: Graphics, H: High disk, T: Cheap, D: Density, R: RAM, M: Main choice, C: Compute, P: Gfx/Pics, X: Extreme Memory, Z: Extreme memory & compute, A: ARM, U: Bare Metal
- Summary:
  - EC2: VMs in the cloud
  - Pricing: On demand, reservered, spot (not charged for remainder of time is killed by AWS), dedicated

## EC - Launching instances

-
