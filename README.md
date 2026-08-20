# RF Front-End Selection for STM32H7 SDR Project

## Goal

Design a multiband HF SDR receiver based on STM32H7B3.

Target bands:

- 80m
- 40m
- 30m
- 20m
- 17m
- 15m
- 10m

The original idea was to use a relay-switched BPF bank and keep the rest of the hardware broadband.

```text
Antenna
   |
Relay-selected BPF
   |
RF Front-End
   |
ADC
   |
Undersampling
   |
DSP / DDC
```

This allows changing only:

- selected BPF
- ADC sampling frequency
- DSP parameters

while leaving the RF front-end unchanged.

---

# Two possible SDR architectures

## Architecture 1: Tayloe Detector

```text
Antenna
   |
BPF
   |
Tayloe Detector
   |
Audio / Baseband I/Q
   |
ADC
   |
DSP
```

Advantages:

- excellent dynamic range
- proven architecture
- simple DSP
- low ADC speed required

Disadvantages:

- local oscillator required
- frequency-specific architecture
- less flexible than direct RF sampling

---

## Architecture 2: Direct RF Undersampling

```text
Antenna
   |
BPF
   |
Broadband LNA
   |
ADC
   |
Undersampling
   |
DSP
```

Advantages:

- simpler signal chain
- no Tayloe detector
- no quadrature switching
- same hardware for all bands

Disadvantages:

- ADC performance becomes critical
- RF front-end must be carefully designed
- dynamic range depends heavily on ADC

---

# Why compare Tayloe receivers?

Before selecting the RF front-end for direct sampling, two existing Tayloe-based receivers were characterised:

1. SoftRock Lite II
2. QRP Labs Receiver

The idea was to establish a baseline.

If a future:

```text
BPF
+
LNA
+
STM32H7 ADC
```

cannot outperform the existing receivers, then there is little reason to replace them.

---

# Test Setup

The setup used during measurements is shown below.

The RF signal generator was connected to the receiver RF input.

The receiver output was observed on a Hantek oscilloscope and output voltage was measured while varying RF input level.

![Measurement Setup](measurement.jpeg)

The QRP Labs receiver was configured with LO around 7.1 MHz.

When RF was offset from LO, the output frequency moved accordingly and appeared as an audio-frequency beat note. This confirmed correct Tayloe operation.

Example:

```text
7.100 MHz RF -> ~150 Hz output

7.120 MHz RF -> ~20 kHz output

7.150 MHz RF -> ~50.6 kHz output
```

---

# Comparison Results

## SoftRock Lite II

The SoftRock Lite II has an onboard crystal LO fixed at **7.05 MHz**.

![SoftRock Lite II](softrock.jpeg)

| RF Input | Output |
|-----------|----------|
| -70 dBm | 5 mVpp |
| -62 dBm | 15 mVpp |
| -48 dBm | 42 mVpp |
| -42 dBm | 82 mVpp |
| -30 dBm | 320 mVpp |
| -20 dBm | 1.01 Vpp |
| -13 dBm | 2.14 Vpp |

---

## QRP Labs Tayloe

The QRP Labs receiver was driven by an external **SI5351A** module used as LO at **7.1 MHz**, controlled via I2C from an STM32.

![SI5351A as LO](si5351a.jpeg)

The output transformers were **not fitted**: they are only needed when connecting to a soundcard. For this SDR project the I/Q baseband is fed directly into the STM32H7 ADC, so the transformers are unnecessary.

![QRP Labs Receiver (no output transformers)](qrp_labs_rx.jpeg)

| RF Input | Output |
|-----------|----------|
| -70 dBm | not detectable |
| -62 dBm | not detectable |
| -48 dBm | 50 mVpp |
| -42 dBm | 90 mVpp |
| -30 dBm | 1.32 Vpp |
| -20 dBm | 2.91 Vpp |
| -13 dBm | 2.98 Vpp |

---

# Analysis

## Weak Signals

Reference:

```text
S9 = -73 dBm
```

Important observations:

```text
-70 dBm
```

SoftRock:

```text
5 mVpp
```

QRP Labs:

```text
not detectable
```

Likewise:

```text
-62 dBm
```

SoftRock still produces a measurable output:

```text
15 mVpp
```

while QRP Labs does not.

### Conclusion

For weak signals:

```text
SoftRock Lite II is clearly superior.
```

---

## Medium Signals

For:

```text
-48 dBm
-42 dBm
-30 dBm
```

QRP Labs produces slightly higher to much higher output voltage.

Example:

```text
-30 dBm

SoftRock = 320 mVpp

QRP Labs = 1.32 Vpp
```

### Conclusion

For medium signals:

```text
QRP Labs provides significantly more gain.
```

---

## Strong Signals

QRP Labs reaches:

```text
2.91 Vpp @ -20 dBm
```

and

```text
2.98 Vpp @ -13 dBm
```

indicating obvious compression.

SoftRock still increases output significantly between these two levels.

### Conclusion

QRP Labs appears to compress around:

```text
≈ 3 Vpp output
```

---

# Current Ranking

## Weak Signals

Winner:

```text
SoftRock Lite II
```

Reason:

- detects signals down to -70 dBm
- detects signals at -62 dBm
- better behaviour near S9 level

---

## Medium Signals

Winner:

```text
QRP Labs Tayloe
```

Reason:

- considerably more output voltage
- stronger output drive

---

# Next Step

A Chinese broadband LNA based on:

```text
TQP3M9037
```

will be measured next.

Cost:

```text
13 EUR
```

The same measurement procedure will be repeated and compared against:

- SoftRock Lite II
- QRP Labs Tayloe

Measurement table prepared for TQP3M9037:

| RF Input | SoftRock | QRP Labs | TQP3M9037 |
|-----------|-----------|-----------|-----------|
| -70 dBm | 5 mVpp | n.d. | |
| -62 dBm | 15 mVpp | n.d. | |
| -48 dBm | 42 mVpp | 50 mVpp | |
| -42 dBm | 82 mVpp | 90 mVpp | |
| -30 dBm | 320 mVpp | 1.32 Vpp | |
| -20 dBm | 1.01 Vpp | 2.91 Vpp | |
| -13 dBm | 2.14 Vpp | 2.98 Vpp | |

---

# Candidate if TQP3M9037 fails

The current preferred fallback is:

```text
Nooelec LaNA HF
```

because it is specifically designed for:

```text
50 kHz - 150 MHz
```

and therefore matches the requirements of a broadband HF SDR much better than microwave-oriented MMIC amplifiers.
