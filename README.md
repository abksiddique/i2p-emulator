# I2P Testnet Emulator

A controlled, desktop-deployable I2P testnet emulation platform for
cybersecurity research, cyber warfare training exercises, and darknet
threat analysis. Runs real I2P Java router instances inside isolated
Linux network namespaces on a single Ubuntu host.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Ubuntu%2022.04%2B-orange.svg)]()
[![Python](https://img.shields.io/badge/Python-3.8%2B-green.svg)]()

## What this is

I2PShield lets security researchers, law enforcement analysts, and students
experiment with how the Invisible Internet Project (I2P) behaves — without
touching live I2P infrastructure. Tunnels form, floodfill routers coordinate,
and eepsites operate exactly as they do on the real network, in a fully
isolated lab environment.

> **Built on:** [boutros-farah/i2p-emulator](https://github.com/boutros-farah/i2p-emulator)
> with packaging, UI, and usability enhancements.

## Download and Install

### Recommended — install the .deb package

Download the latest `.deb` from the
[Releases page](https://github.com/abksiddique/i2p-emulator/releases/latest)
then run:

```bash
sudo apt install ./i2p-emulator_1.0.0_all.deb
```

Launch from the Applications menu or run:

```bash
i2p-emulator
```

### From source

```bash
git clone https://github.com/abksiddique/i2p-emulator.git
cd i2p-emulator

sudo apt install python3 python3-pyqt5 openjdk-17-jre-headless iproute2 psmisc
pip3 install --break-system-packages pycountry countryinfo

python3 working-gui.py
```

## Requirements

- Ubuntu 22.04 LTS, 24.04 LTS, or 26.04
- 4 GB RAM minimum
- Internet connection for first deploy (downloads I2P router JAR ~30 MB)
- Linux kernel with network namespace support (standard on all Ubuntu releases)

## First-run workflow

1. Launch — the setup wizard opens automatically on fresh install
2. **Topology Builder tab** — sample 7-router topology is pre-loaded
3. Click **Validate** — confirms topology is correct
4. Click **Export Files** — generates deployment tables
5. Click **Deploy Network** — downloads I2P, creates namespaces, starts routers
6. Wait 2–4 minutes — routers appear in the **Fleet** tab as they start

## Default topology

| Location | Subnet | Routers | Floodfill |
|----------|--------|---------|-----------|
| Lebanon (Beirut) | 45.10.1.0/29 | 3 | 1 |
| Germany (Berlin) | 91.80.1.0/29 | 2 | 1 |
| France (Paris)   | 185.20.1.0/29 | 2 | 0 |

## Features

- Isolated I2P routers in Linux network namespaces
- Public-style IPv4 addressing across distinct /16 families
- GUI fleet monitoring with live peer and tunnel statistics
- Geographic network map (Leaflet) with router markers
- Churn scenario testing (mild / moderate / aggressive presets)
- Measurement campaigns with CSV and JSON export
- Five-phase authoritative tunnel-hop truth pipeline
- First-run setup wizard
- Single .deb package — one command to install

## Repository structure

```
.
├── working-gui.py                              # PyQt5 control centre (19,813 lines)
├── setup-i2p-emulator.sh                      # Deployment script (2,415 lines)
├── topology_model.py                           # Topology validation
├── topology.sample.json                        # Default 7-router topology
├── topology.public-any-location.template.json  # Public-IP topology template
├── phase5_backend.py                           # Truth pipeline backend
├── import_java_authoritative_truth.py          # Java JSONL importer
├── export_deployment_tables.py                 # Deployment TSV exporter
├── export_subnet_tables.py                     # Subnet TSV exporter
├── build_topology_manifest.py                  # Manifest builder
├── run_phase5b_normalization.py                # Normalisation worker
├── run_phase5c_scan.py                         # Scan worker
├── run_phase5d_change_detection.py             # Change detection worker
├── DEBIAN/                                     # Package build scripts
│   ├── control
│   ├── postinst
│   ├── prerm
│   └── postrm
└── README.md
```

## Authoritative tunnel-hop truth

For research requiring exact tunnel-hop knowledge, the platform integrates
with a patched I2P router fork
([boutros-farah/i2p.i2p](https://github.com/boutros-farah/i2p.i2p),
branch `authoritative-hop-writer`) that emits JSONL tunnel events directly
from inside the Java router.

Only records with `source_mode = java-router-authoritative` and
`truth_level = ground-truth` are treated as canonical truth.

## Validation results

| Metric | Result |
|--------|--------|
| Routers deployed | 7 |
| All routers active | ✓ within 4 minutes |
| Baseline probes (best run) | 8/8 completed |
| Mean root latency | 81.5 ms |
| Mean proxy latency | 4.5 ms |
| Canonical truth events | 598 |
| Path change events | 144 |

## Runtime data directories

| Directory | Contents |
|-----------|----------|
| `~/i2p-emulator-data/` | Generated topology TSV files |
| `~/i2p-testnet-N/` | Testnet deployment and router configs |
| `~/i2p-gui/logs/hop_history/` | Observed path records |
| `~/i2p-gui/logs/hop_truth/` | Authoritative truth records |

## Uninstall

```bash
sudo apt remove i2p-emulator
```

Runtime data in `~/i2p-emulator-data/`, `~/i2p-testnet-*/`, and
`~/i2p-gui/` is preserved. Remove manually if no longer needed.

## Disclaimer

This platform runs in a completely isolated lab environment and does not
connect to or affect the live I2P network. It is intended for legitimate
security research, academic study, and training purposes only. Users are
responsible for ensuring their use complies with applicable laws and
institutional policies.

## License

MIT — see [LICENSE](LICENSE)

Original emulator: [boutros-farah/i2p-emulator](https://github.com/boutros-farah/i2p-emulator)
