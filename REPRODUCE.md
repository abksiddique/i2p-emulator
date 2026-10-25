# Reproducing the CSET 2026 Results

This guide reproduces the evaluation in *Garlic in a Box: I2P Network
Emulator* (CSET 2026): a 50-router isolated testnet across five geographic
locations, and a five-trial reputation-injection attack on a victim router.

## 1. Host requirements

- Ubuntu 22.04 LTS, 24.04 LTS, or 26.04
- At least 24 GB RAM free (the 50-router fleet settles near 21 GB,
  about 417 MB per router)
- Linux kernel with network namespace support (standard on Ubuntu)
- The reference runs were performed on an Ubuntu 24.04 guest with
  48 GB RAM, 12 vCPUs, and 200 GB disk

## 2. Install

```bash
sudo apt install ./i2p-emulator_1.0.0_all.deb
i2p-emulator
```

Or run from source as described in the README.

## 3. Build the 50-router topology

Five locations, ten routers each, one floodfill per location (five
floodfills total):

| Location        | Subnet          | Routers | Floodfill |
|-----------------|-----------------|---------|-----------|
| Lebanon (LB)    | 45.10.1.0/27    | 10      | 1         |
| Germany (DE)    | 91.80.1.0/27    | 10      | 1         |
| France (FR)     | 185.20.1.0/27   | 10      | 1         |
| Japan (JP)      | 23.10.1.0/27    | 10      | 1         |
| United States   | 150.20.1.0/27   | 10      | 1         |

In the Topology Builder, enter these five locations and generate the
topology JSON, or start from `topology.sample.json` and extend it to
50 routers. The victim is `r10` (IP 45.10.1.11). The five attackers are
`r45` through `r49`, placed in the US cluster, separate from the victim.

## 4. Deploy and let the network emerge

Deploy from the GUI, then wait about 25 minutes, undisturbed. Do not pause
or restart routers during emergence; a healthy fleet needs the full window.
When emergence completes, every router shows ACTIVE in the Fleet view, and
the victim reports close to 49 known peers.

## 5. Run the reputation-injection attack (five trials)

Open the Scenarios panel, select the reputation-injection template, set the
attacker set to `r45` through `r49`, and run five independent trials.

Each trial follows the procedure in Algorithm 2 of the paper:

1. Clean: zero the attacker bonuses in the victim's profiles, settle 10 min
2. Baseline: stop the victim, snapshot profiles, restart
3. Gate: confirm all attacker bonuses read zero, else abort
4. Attack: apply the behavioral config, inject speedBonus and capacityBonus
   of 9999, start, settle 10 min
5. Attack snapshot: stop the victim, copy profiles, restart
6. Analyze: resolve identity hashes and compute per-metric ratios

The anti-contamination gate must pass on every trial. If a baseline still
carries injected values from a prior run, the trial aborts by design.

## 6. Where the results land

Aggregated results across the five trials are written to `AGGREGATE.json`.
<!-- CONFIRM: set this to the actual output path your pipeline writes,
     e.g. ~/i2p-gui/logs/... , and name the script that produces it -->

## 7. Expected results

| Metric                          | Attackers        | Controls        |
|---------------------------------|------------------|-----------------|
| speedBonus                      | 9999.0 +/- 0.0   | 0.0 +/- 0.0     |
| capacityBonus                   | 9999.0 +/- 0.0   | 0.0 +/- 0.0     |
| tunnelPeakThroughput (KBps)     | 0.127 +/- 0.000  | 0.210 +/- 0.072 |
| lifetimeAgreedTo                | 10.76 +/- 0.20   | 7.80 +/- 1.84   |
| lifetimeRejected                | 265.8 +/- 21.2   | 77.0 +/- 16.5   |
| responseMs                      | 21.71 +/- 4.82   | 16.52 +/- 1.29  |

Key ratios:

- Selection stage: injection guarantees eligibility in all five trials
- Tunnels agreed-to: attackers chosen 1.38x more than controls
- Tunnels rejected: attackers reject 3.45x more than controls
- Throughput: 0.60x, the behavioral lever is weak under light testbed load

Small run-to-run variation is expected on the measured fields. The injected
bonus fields are deterministic and should read exactly 9999 with zero
variance.

## Notes

- Keep the testnet isolated. The filesystem bootstrap means no router needs
  the public reseed servers; confirm no external I2P traffic leaves the host.
- Run each trial end to end without pausing, or the coalescing timer may
  overwrite the injected values.
