# PAM-N Signal Integrity Automator

**Automated signal integrity analysis for Automotive Ethernet PAM-N signals using the R&S RTP oscilloscope.**

Developed by R&S Application Engineering to streamline 2.5 / 5 / 10 GBASE-T1 measurements that would otherwise require many manual steps on the instrument.

---

## Overview

This tool provides a graphical interface that controls an R&S RTP oscilloscope over a network connection (TCPIP/VISA). It automates signal configuration, eye diagram setup, and advanced jitter decomposition — replacing repetitive manual instrument operation with a single button press.

---

## Files

| File | Purpose |
|---|---|
| `PAM_AE_SignalIntegrity_GUI.py` | Main application — run this to launch the tool |
| `PAM_AE_SignalIntegrity_Backend.py` | Instrument automation functions (SCPI over VISA) |

---

## Requirements

**Python 3.10 or higher**

Install dependencies:

```bash
pip install RsInstrument
```

Tkinter is included with standard Python on Windows. No other packages required.

---

## How to Run

```bash
python PAM_AE_SignalIntegrity_GUI.py
```

---

## Features

### Signal Configuration
- Selectable channel pair: CH1+CH3 or CH2+CH4
- Creates a DIFF1 differential signal automatically
- Technology presets: 2.5 / 5 / 10 GBASE-T1
- Configurable vertical and horizontal scaling
- Factory preset + full reconfiguration in one click

### Eye Diagram
- **Software CDR**: Whole eye, specific eye (Eye1/2/3), or selected transition
- **Hardware CDR**: CDR trigger lock with optional zoom window
  - Zoom coordinates configurable in nanoseconds
  - Zoom window automatically removed and recreated on each run

### Jitter Analysis
- **Quick Start**: Fully automatic — instrument detects bit rate and enables all components
- **Manual Components**: Select any combination of DDJ, RJ, TJ, DJ, PJ, DCD
- **PAM-N Results**: Transition-dependent histograms for all six PAM-4 rising transitions (0→1, 0→2, 0→3, 1→2, 1→3, 2→3)

### Workflow Intelligence
- Switching **Jitter → Eye**: jitter results display turned off automatically before eye setup
- Switching **HW CDR → SW CDR**: zoom window removed, trigger reset to EDGE automatically
- Switching **Quick Start → Manual/PAM-N**: Step Response, DDJ Bathtub, DDN Bathtub disabled automatically
- Heavy operations (CLEJitcomp, CATegory:ADD) use a polling strategy with live progress ticks — no VISA timeout regardless of instrument computation time

---

## Instrument Connection

1. Connect the R&S RTP oscilloscope to the same network as your PC
2. Find the instrument IP address on the scope: **Setup → Network**
3. Enter the IP in the **Connection** tab and click **Connect**

Default IP pre-filled: `10.103.34.23` — change this to match your instrument.

---

## Building a Standalone Executable

To distribute the tool without requiring Python to be installed:

```bash
pip install cx_freeze
python setup.py build
```

The `build/` folder will contain `SignalIntegrityAutomator.exe` and all dependencies. Use `setup.py` from the repository.

---

## Repository Structure

```
pam_signal_integrity_autom/
├── PAM_AE_SignalIntegrity_GUI.py       # GUI entry point
├── PAM_AE_SignalIntegrity_Backend.py   # SCPI automation backend
├── setup.py                            # cx_Freeze build configuration
└── README.md                           # This file
```

---

## Contributing / Updating

This project follows a standard Git workflow. See the [Git usage guide](#git-usage) below.

### Git Usage

**One-time setup — clone the repository to your PC:**

```bash
git clone https://code.rsint.net/inneRSource/python/pam_signal_integrity_autom
cd pam_signal_integrity_autom
```

**Every time you make a change:**

```bash
# 1. Get the latest version before starting work
git pull

# 2. Edit your files

# 3. Stage the files you changed
git add PAM_AE_SignalIntegrity_Backend.py PAM_AE_SignalIntegrity_GUI.py

# 4. Commit with a short description of what changed
git commit -m "Brief description of change"

# 5. Push to GitLab
git push
```

**Commit message conventions:**

| Prefix | Use for |
|---|---|
| `fix:` | Bug fixes |
| `feat:` | New features |
| `refactor:` | Code cleanup with no behaviour change |
| `docs:` | README or comment updates |

Examples:
```
fix: resolve zoom -222 error when coming from jitter
feat: add TJ@BER component to manual jitter selection
docs: update README with network setup instructions
```

---

## SCPI Reference

All SCPI commands are sourced from the **R&S RTP Oscilloscope User Manual**.

Key command areas used:
- `ADVJitter1:*` — Advanced jitter decomposition (instance 1)
- `EYE1:*` — Eye diagram configuration
- `TRIGger1:TYPE CDR` — Hardware CDR trigger
- `LAYout:ZOOM:ADD / REMove` — Zoom window management
- `SIGNalconfig:SETup:ADD` — Technology standard preset

---

## Known Instrument Behaviour

- The HW CDR zoom window may take a few seconds to render after being created — this is normal instrument pipeline behaviour, not a code issue.
- `CLEJitcomp` (jitter decomposition reset) can take 60–120 seconds depending on signal complexity. The log shows live progress ticks every 15 seconds.
- 10GBASE-T1 at ~3.8 GBd symbol rate may not lock with HW CDR — SW CDR is recommended for this standard.