# ECG Analog Front-End Filter: LTspice Simulation

Design and verification of a passive RC band-pass filter for ECG signal
conditioning (0.5 to 40 Hz), built up from first principles in LTspice.

## 1. Objective and specification

| Parameter | Target |
|---|---|
| Passband | 0.5 Hz to 40 Hz (-3 dB points) |
| Passband flatness | within about 1 dB between 1 Hz and 10 Hz |
| Out-of-band rejection | attenuate 50 Hz mains interference |
| Topology | passive first-order RC low-pass + high-pass |

## 2. Method

Each building block was designed by hand, simulated, and compared
with the calculation before moving to the next stage.

| # | Circuit | Design equation |
|---|---|---|
| 01 | Series divider | V_out = V x R2 / (R1 + R2) |
| 02 | Parallel resistors | I_total = I1 + I2 (Kirchhoff's current law) |
| 03 | RC step response | tau = R x C |
| 04 | RC low-pass | fc = 1 / (2 pi R C) |
| 05 | RC high-pass | fc = 1 / (2 pi R C) |
| 06 | Band-pass (cascade) | low-pass fc = 40 Hz, high-pass fc = 0.5 Hz |
| 07, 08 | Noise rejection test | 1 Hz signal + 50 Hz interference |

## 3. Results: calculation vs simulation

| Test | Calculated | Simulated |
|---|---|---|
| Series: midpoint voltage, current (10 V, 1k + 3k) | 7.5 V, 2.5 mA | 7.5 V, 2.5 mA |
| Parallel: I(R1), I(R2), I(source) (10 V, 1k || 3k) | 10, 3.33, 13.33 mA | 10, 3.33, 13.33 mA |
| RC step: voltage at t = tau (1k, 1 uF, 5 V step) | 3.16 V at 1 ms | TBD |
| Low-pass cutoff (1k, 1 uF) | 159 Hz | TBD |
| High-pass cutoff (318k, 1 uF) | 0.50 Hz | TBD |
| Band-pass lower -3 dB point | about 0.5 Hz | TBD |
| Band-pass upper -3 dB point | about 40 Hz | TBD |
| 50 Hz attenuation, R2 = 3.98k | about -4.1 dB | TBD |
| 50 Hz attenuation, R2 = 15.9k | about -14.2 dB | TBD |

### 01 Series circuit
![Series](screenshots/01_series.png)
CAPTION 01

### 02 Parallel circuit
![Parallel](screenshots/02_parallel.png)
CAPTION 02

### 03 RC step response
![RC](screenshots/03_RC_transient.png)
CAPTION 03

### 04 Low-pass filter
![Low-pass](screenshots/04_lowpass_AC.png)
CAPTION 04

### 05 High-pass filter
![High-pass](screenshots/05_highpass_0p5Hz.png)
CAPTION 05

### 06 Band-pass filter
![Band-pass](screenshots/06_bandpass_AC.png)
CAPTION 06

### 07 Noise test, R2 = 3.98k
![Noise test](screenshots/07_bandpass_noise_test.png)
CAPTION 07

### 08 Noise test, R2 = 15.9k
![Noise test 10 Hz](screenshots/08_bandpass_10Hz_test.png)
CAPTION 08

## 4. Discussion

- The simulated results agree with the hand calculations (see the table).
- The passband gain is slightly below 0 dB. At 1 Hz, the 0.5 Hz
  high-pass stage alone gives |H| = 0.89 (-1 dB), and the peak of the
  1 Hz test signal is about 0.9 V.
- First-order stages roll off at only -20 dB/decade. The 50 Hz
  interference lies close to the 40 Hz cutoff and is attenuated by only
  about 4 dB. Lowering the low-pass cutoff to 10 Hz gives about 14 dB,
  but it also attenuates ECG components above 10 Hz.
- The two stages interact slightly, because the second stage loads the
  first. The 318k load on the 3.98k stage makes this error small (about 1%).

## 5. Limitations

- No gain: a real ECG (about 1 mV) needs an instrumentation amplifier
  and gain stages before filtering.
- First-order rolloff is not enough for 50 Hz rejection. A notch or
  higher-order active filter is needed.
- Ideal components only: no tolerances, op-amp noise, input impedance,
  or patient-safety isolation (IEC 60601-1) are modelled.
- No Monte Carlo or temperature analysis of component tolerance.

## 6. Next steps

1. Add an op-amp buffer and gain stage, and a 2nd-order active filter.
2. Add a 50 Hz twin-T notch filter.
3. Run Monte Carlo analysis (1% resistors, 10% capacitors).
4. Repeat the filtering in Python (SciPy) on real MIT-BIH ECG data and
   validate R-peak detection against expert annotations.

## 7. Repository structure

```
circuits/       LTspice .asc files (01 to 08)
screenshots/    simulation results
README.md
```

Tool: LTspice (Analog Devices). Open any `.asc` file and press Run.
