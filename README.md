# PSD-Based Micro-Cantilever Characterization System

Design and implementation of a transimpedance and differential amplifier front-end for optical-lever deflection sensing using a Hamamatsu S3932 Position Sensitive Detector (PSD). The system converts micro-cantilever deflection into a clean, processable electrical signal for extracting **resonant frequency** and **damping ratio**.

## Overview

A laser reflects off a micro-cantilever onto the PSD. Cantilever deflection shifts the laser spot position on the detector, producing two photocurrents whose relative magnitudes encode position. The PSD's junction capacitance interacts with the amplifier feedback network, causing high-frequency instability if left uncompensated. This project addresses that with a stability-compensated transimpedance amplifier (TIA) followed by a differential amplifier stage, validated in SPICE and on hardware.

The project spans the full measurement chain — from the optical excitation and readout of the cantilever, through analog signal conditioning, to digital acquisition and processing — culminating in resonant frequency and damping ratio extraction.

## End-to-End Project Flow

```
 STAGE 1 — Optical Setup
 ────────────────────────
 Laser Diode ──► Beam Alignment/Focusing Optics ──► Micro-Cantilever Surface
                                                              │
                                            (Cantilever deflects due to
                                             mechanical excitation /
                                             external stimulus / thermal noise)
                                                              │
                                              Reflected Beam Shifts Position
                                                              │
                                                              ▼
 STAGE 2 — Optical-to-Electrical Transduction
 ─────────────────────────────────────────────
                                    PSD (Hamamatsu S3932)
                                    Two photocurrent outputs: I1, I2
                                    (ratio encodes spot position;
                                     sum encodes beam intensity)
                                                              │
                                                              ▼
 STAGE 3 — Analog Front-End (Amplifier Design)
 ───────────────────────────────────────────────
        Transimpedance Amplifier (TIA) x2  (I1,I2 -> V1,V2)
              - Virtual-ground biasing at PSD electrodes
              - Feedback capacitor (Cf) compensates PSD junction
                capacitance -> stability at high frequency
              - Input bias current considered as error source
                                                              │
                              ┌───────────────────────────────┤
                              ▼                                ▼
              Difference Amp (V1-V2)              Summing Amp (V1+V2)
                 [position signal]                  [intensity signal]
                              │                                │
                              └───────────────┬────────────────┘
                                               ▼
                              Buffer / Scaling Stage (impedance isolation)
                                               │
                                               ▼
                         Normalization: (V1-V2)/(V1+V2)
                     (analog divider circuit, OR passed through
                      unnormalized for digital normalization)
                                               │
                                               ▼
 STAGE 4 — Data Acquisition
 ────────────────────────────
                          ADC / DAQ (sampled at rate >> cantilever
                          resonant frequency, anti-alias filtered)
                                               │
                                               ▼
 STAGE 5 — Digital Signal Processing (DSP)
 ────────────────────────────────────────────
        Digitized Position Signal x(t)
                     │
                     ├─► Digital Normalization (if not done in analog): (V1-V2)/(V1+V2)
                     ├─► Windowing + FFT / Power Spectral Density
                     │        -> identify resonant frequency f0 (spectral peak)
                     ├─► Time-domain fit of decaying oscillation
                     │        (or -3dB bandwidth of resonance peak)
                     │        -> extract damping ratio ζ / quality factor Q
                     └─► Filtering (low-pass / band-pass) to reject
                              amplifier noise and out-of-band artifacts
                     │
                     ▼
        Characterized Cantilever Parameters:
        Resonant Frequency (f0) & Damping Ratio (ζ)
```

## Key Design Considerations

### Optical Setup
- **Optical Lever Geometry**: Laser spot placement near the cantilever's free tip maximizes sensitivity, since local slope (not displacement) at the incidence point drives the PSD signal, not absolute tip displacement.
- **Beam Alignment**: Laser must be centered on the PSD's active area at rest to maximize linear range and avoid saturation of either electrode.
- **Excitation Source**: Cantilever deflection may be driven by piezo actuation, thermal/Brownian motion, or an external stimulus, depending on the characterization method used.

### Analog Front-End (Amplifier Design)
- **Transimpedance Stage**: Virtual-ground biasing at the PSD electrodes; feedback capacitor (Cf) sized to compensate detector junction capacitance and maintain phase margin/bandwidth.
- **Bias Current Effects**: Op-amp input bias current is analyzed as a source of position measurement error.
- **Differential Stage**: Extracts position (V1−V2) and total intensity (V1+V2) while rejecting common-mode noise.
- **Normalization**: (V1−V2)/(V1+V2) removes dependence on laser intensity fluctuations, implemented in analog hardware and/or digitally in post-processing.

### Data Acquisition & DSP
- **Sampling Rate**: Chosen well above the cantilever's expected resonant frequency to satisfy Nyquist and preserve waveform shape for damping analysis.
- **Anti-Aliasing**: Analog low-pass filtering before the ADC to prevent high-frequency noise/PSD artifacts from folding into the band of interest.
- **Resonant Frequency Extraction**: FFT/PSD of the position signal; f0 identified as the dominant spectral peak.
- **Damping Ratio Extraction**: Either from the decay envelope of a time-domain impulse/ring-down response, or from the −3 dB bandwidth of the resonance peak (Q-factor method).

## Repository Structure

```
├── optical-setup/       # Alignment notes, beam path diagrams, mounting details
├── simulation/          # SPICE models and simulation files (TIA, differential stage)
├── hardware/             # Schematics, PCB layout, BOM
├── firmware/             # (if applicable) microcontroller/DAQ interface code
├── dsp/                  # FFT / PSD / curve-fitting scripts for f0 and ζ extraction
├── analysis/             # Post-processed data, plots, characterization results
├── docs/                 # Design notes, derivations, reference datasheets
└── README.md
```
*(Adjust folder names above to match your actual repo layout.)*

## Simulation

Frequency response, stability margin, and noise performance of the TIA and differential stages were validated in SPICE prior to hardware implementation, confirming adequate phase margin after Cf compensation.

## Hardware

The amplifier chain was built and tested with the Hamamatsu S3932 PSD to confirm signal integrity and stability at the frequencies relevant to cantilever resonance measurement.

## DSP / Analysis

The digitized, normalized position signal is processed to extract:
- **Resonant frequency (f0)** via FFT/power spectral density of the position signal.
- **Damping ratio (ζ)** via ring-down decay fitting or resonance peak bandwidth (Q-factor).

## Results

Extracted resonant frequency and damping ratio of the micro-cantilever from the conditioned, digitized PSD output, demonstrating a stable, low-noise, end-to-end signal chain from optical excitation through DSP-based characterization.

## Requirements

- Hamamatsu S3932 PSD
- Laser diode + alignment optics
- Op-amps (specify part numbers used)
- SPICE simulator (e.g., LTspice/PSpice)
- ADC/DAQ or oscilloscope for data capture
- Python/MATLAB (or similar) for FFT and curve-fitting in DSP stage

## License

Specify your license here (e.g., MIT).
