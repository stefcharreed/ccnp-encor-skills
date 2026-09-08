---
name: ccnp-wireless-signals-and-modulation
description: >
  Use this skill when troubleshooting or designing wireless RF at Layer 1 —
  signal characteristics, power math, and modulation. Invoke when the user asks
  about: RF, radio frequency, wave propagation, frequency, hertz, band, channel,
  channel width, channel separation, signal bandwidth, center frequency,
  spectral mask, non-overlapping channels, channels 1 6 11, phase, in phase,
  out of phase, wavelength, lambda, amplitude, RF power, decibel, dB, dBm, dBi,
  dBd, isotropic antenna, dipole, Law of 3s, Law of 10s, EIRP, link budget,
  free space path loss, FSPL, RSSI, sensitivity level, noise floor, SNR,
  signal-to-noise ratio, carrier signal, modulation, demodulation, narrowband,
  spread spectrum, DSSS, OFDM, OFDMA, QAM, 802.11 amendments, Wi-Fi 6, Wi-Fi 6E,
  SISO, MIMO, spatial multiplexing, spatial streams, transmit beamforming, TBF,
  maximal-ratio combining, MRC, dynamic rate shifting, DRS.
---

## Purpose
This is the physics layer under every wireless problem: how an RF signal is
described (frequency, phase, wavelength, amplitude), how its power is measured
and budgeted end to end (dB, dBm, EIRP, free space path loss, RSSI, SNR), and
how data is actually carried on it (modulation, spread spectrum, MIMO, and
dynamic rate shifting). **Almost every "the Wi-Fi is slow" complaint resolves to
a number in this chapter — usually SNR or RSSI at the cell edge.**

## Key Concepts

### Wave propagation
- A transmitter feeds alternating current into an antenna; the current produces
  electromagnetic waves that propagate outward. At the far end the process
  reverses — the arriving waves **induce an electrical signal** in the
  receiver's antenna. If everything works, the received signal is a reasonable
  copy of the transmitted one.
- The **idealistic (isotropic) antenna** radiates equally in every direction.
  **It does not exist** — it is a reference point. Real antennas are shaped to
  limit and direct where the waves go.

### Frequency
- **Frequency** is the number of complete up-and-down **cycles** the signal
  makes in one second. A cycle can be measured from the rise through the center
  line, down through it, and back up to it — or simply peak to peak. Wherever
  you start, the signal must complete the full sequence back to its starting
  position.
- **Hertz (Hz) is nothing more than one cycle per second.** Four cycles in one
  second = 4 Hz.
- Units scale by thousands: **kHz** = 1,000 Hz · **MHz** = 1,000,000 Hz ·
  **GHz** = 1,000,000,000 Hz.
- The continuous spectrum runs from 0 Hz to about 10²² Hz: subsonic, sound,
  then the **radio frequencies** (low-frequency radio, AM, shortwave, TV/FM,
  microwave and radar), then infrared, visible light, ultraviolet, X-rays,
  gamma rays, cosmic rays. **Wireless LANs live in the microwave/radar region —
  2.4, 5 and 6 GHz.**

### Bands, channels, and why 2.4 GHz is different
- A **band** is a contiguous frequency range; it is divided into **channels**,
  each numbered and assigned a specific frequency by a national or
  international standards body so they can be used consistently everywhere.
- The 2.4 GHz band has **14 channels**, spaced **0.005 GHz (5 MHz)** apart —
  channel 1 at 2.412 GHz up to channel 13 at 2.472 — **except channel 14** at
  2.484. That spacing is the **channel separation**, or **channel width**.
- **An RF signal is not infinitely narrow.** It spills above and below a
  **center frequency**, occupying neighbouring frequencies. The center
  frequency defines the channel's location; the actual frequency range the
  signal needs is the **signal bandwidth**. A 22 MHz-bandwidth signal is bounded
  11 MHz above and below its center frequency.
- Devices use a **spectral mask** to ignore parts of a signal falling outside
  the bandwidth boundaries.
- **The 2.4 GHz problem:** channels are **5 MHz** wide but Wi-Fi signals have a
  **22 MHz** bandwidth — slightly wider than four channels. **Adjacent channel
  numbers are not spaced far enough apart to be non-overlapping**, so signals
  must be placed on more distant channels, which limits how many channels are
  usable in the band.
- **5 and 6 GHz do not have this problem:** channels are spaced every **20 MHz**
  to support signals about 20 MHz wide, so **every channel can be used without
  interfering with its neighbours**, maximising available channels.
- *Why would a standards body number channels that overlap?* Channels are
  defined for a specific use at a point in time; a later technology may reuse
  the same band with signals needing more bandwidth than the original numbering
  anticipated. **That is exactly what happened to 2.4 GHz Wi-Fi.**

### Phase
- **Phase** is a measure of shift in time relative to the start of a cycle,
  measured in degrees: **0° at the start, 360° for one complete cycle, 180°
  halfway through.** Because the signal is cyclic, think of phase travelling
  around a circle again and again.
- Two identical signals produced at exactly the same time are **in phase**; one
  delayed relative to the other is **out of phase**.
- **In-phase signals add together; signals 180° out of phase cancel each other
  out.** This one sentence is the mechanism behind both transmit beamforming
  (deliberately constructive) and multipath fading (accidentally destructive).

### Wavelength
- **Wavelength (λ)** is the physical distance a wave travels over one complete
  cycle.
- RF waves travel at a constant speed — in a vacuum, exactly the speed of light;
  in air, slightly less.
- **Wavelength decreases as frequency increases** — faster cycles cover less
  distance. Wavelength matters for antenna design and placement.
- Feel for the scale: **2.4 GHz ≈ 4.92 in · 5 GHz ≈ 2.36 in · 6 GHz ≈ 1.97 in.**

### Amplitude and absolute power
- **Amplitude** is the height from the top peak to the bottom peak of the
  waveform — the signal's strength.
- Power is measured in **watts**. An AM station may broadcast at 50,000 W and an
  FM station at 16,000 W; a **wireless LAN transmitter runs between 0.1 W
  (100 mW) and 0.001 W (1 mW).**
- Watts and milliwatts are **absolute** power measurements — something has to
  measure exactly how much energy is present.

### Why dB exists
- To compare two transmitters you can subtract or divide, **and the two methods
  disagree.** 1 mW vs 10 mW is 9 mW by subtraction but 10× by division; 10 mW vs
  100 mW is 90 mW by subtraction but again 10×. Worse: 0.00001 mW vs 10 mW is a
  difference of 9.99999 mW — but a ratio of **1,000,000**.
- Absolute power spans a huge range, so a **logarithm** is used to transform the
  exponential range into a linear one — spacing 0.001, 0.01, 0.1, 1, 10, 100,
  1000 evenly.
- **dB = 10(log₁₀P2 − log₁₀P1) = 10 log₁₀(P2/P1)**, where **P2 is the source of
  interest** and **P1 is the reference**. Both forms give identical values; the
  **ratio form is the one used in wireless engineering.**
- **You will not need a calculator or a logarithm on the ENCOR exam** — the
  three laws below are what you actually use.

### The three dB laws
- **Law of Zero:** 0 dB means the two absolute power values are **equal**
  (the ratio is 1, and log₁₀(1) = 0).
- **Law of 3s:** **+3 dB = double** the reference; **−3 dB = half**.
- **Law of 10s:** **+10 dB = ten times**; **−10 dB = one tenth**.
- **When absolute powers multiply, the dB value is positive and is added; when
  they divide, the dB value is negative and is subtracted.** dB values chain, so
  any combination of ×2 and ×10 steps can be expressed as a sum.

### dBm — comparing against a fixed reference
- dB compares two arbitrary values. **dBm** puts a **fixed reference of 1 mW**
  on the bottom of the ratio, so every power level along a path can be converted
  to a common scale and **simply added**.
- **100 mW = 20 dBm.** A 65 dB net loss over the path gives
  **20 dBm − 65 dB = −45 dBm** at the receiver (0.000031623 mW).

### Antenna gain: dBi, dBd, and EIRP
- **An antenna generates no absolute power of its own** — disconnect it and no
  milliwatts come out. So its gain **cannot** be expressed in dBm. Gain is
  measured by comparing the antenna's performance to a **reference antenna** and
  computing a value in dB.
- The usual reference is the **isotropic antenna**, giving gain in **dBi**. It
  is a theoretical tiny point radiating equally in every direction; no physical
  antenna can do that, but its performance is calculable from RF formulas, which
  makes it a **universal reference**.
- **dBd** is referenced to a **dipole** — a real antenna with **2.14 dBi** of
  gain. **Add 2.14 to a dBd figure to convert it to dBi.**
- Cable always loses signal; vendors publish loss in **dB per foot or meter**.
- **EIRP = Tx Power − Tx Cable + Tx Antenna**, expressed in **dBm**.
- 🚨 **EIRP is regulated by government agencies in most countries — a system
  may not radiate above the maximum allowable EIRP.** This is a legal ceiling,
  not a tuning preference.
- **dBm, dBi and plain dB can safely be combined** when computing EIRP. **The
  only exception is dBd** — convert it to dBi first.

### The full link budget
- **Rx Signal = Tx Power − Tx Cable + Tx Antenna − Free Space + Rx Antenna − Rx Cable**
- Gains and losses in dB combine over any number of stages, so if you start with
  transmit power in **dBm**, you just add and subtract along the path.

### Free space path loss
- An RF signal's amplitude decreases as it travels through free space **even
  with no obstacles in the path**.
- **The cause is geometry, not the medium** — signals are degraded even in the
  vacuum of space. The wave expands as a three-dimensional sphere; the same
  energy is spread over an ever-larger surface, so its concentration weakens
  with distance. **Even a tightly focused beam still spreads.**
- **FSPL(dB) = 20 log₁₀(d) + 20 log₁₀(f) + 32.44**, where *d* is distance in
  **kilometers** and *f* is frequency in **megahertz**. **Not required for the
  exam** — it is shown to make two points:
  - **FSPL is an exponential function: the signal falls off quickly near the
    transmitter and more slowly farther away.**
  - **The loss is a function of distance and frequency only.**
- ⚠️ **An indoor path is not negligible.** Distance is given in kilometers and
  indoor clients are usually under 50 m from the AP — but **even at 1 meter,
  free space costs around 46 dB.**
- **FSPL is greater at 5 GHz than 2.4 GHz, and greater at 6 GHz than 5 GHz** —
  as frequency rises, so does loss. **Therefore 2.4 GHz has greater effective
  range than 5 and 6 GHz at equal transmitted signal strength.**

### Power at the receiver: RSSI, sensitivity, noise floor, SNR
- **Transmit side:** EIRP leaving the antenna normally runs 100 mW down to 1 mW
  = **+20 dBm down to 0 dBm**.
- **Receive side:** levels are far smaller — 1 mW down to tiny fractions
  approaching 0 mW = **0 dBm down to about −100 dBm.**
- **RSSI (received signal strength indicator)** is defined in 802.11 as an
  **internal 1-byte relative value from 0 to 255**, 0 weakest and 255 strongest.
  🚨 **It has no useful units, its range varies between manufacturers, and it is
  not standardized across receivers** — an RSSI value can differ from one
  receiver's hardware to another. In practice you see values already converted
  and scaled to dBm.
- **Sensitivity level** is the receiver's threshold dividing intelligible,
  useful signals from unintelligible ones. Above it, the data will probably
  decode; below it, it will not.
- **Noise floor** is the average signal strength of every *other* signal
  received on the same frequency.
- **SNR (signal-to-noise ratio) = signal − noise floor, measured in dB. Higher
  is better.** A −54 dBm signal against a −90 dBm noise floor is **36 dB**; let
  the noise floor rise to −65 dBm and the same signal gives only **11 dB**, at
  which point it may not be usable. **The signal did not change — the noise
  did.** That is why "but the client shows good signal strength" is not an
  answer to a wireless performance complaint.

### Carrying data on the signal
- The plain RF oscillation is the **carrier signal** — constant frequency,
  amplitude and phase. The steady, predictable frequency is what lets a receiver
  tune to it in the first place.
- Naive schemes fail: switching the carrier **on and off** for 1s and 0s means a
  weak or missing signal reads as a long string of 0s; sending only the **upper
  half** of the cycle for a 1 and the lower half for a 0 is impractical to
  receive and very hard to transmit as disjointed alternating cycles.
- **Modulation** alters the carrier according to another source to encode data;
  **demodulation** interprets it at the receiver. Whatever scheme the
  transmitter uses, the receiver must use too.
- Modulation goals: **carry data at a predefined rate**, **be reasonably immune
  to interference and noise**, and **be practical to transmit and receive**.
- 🚨 **A modulation scheme can alter only three attributes** — **frequency**
  (and only by varying slightly above or below the carrier), **phase**, and
  **amplitude**.
- Low-bit-rate signals (AM/FM audio) need little extra bandwidth —
  **narrowband**. Wireless LANs carry high bit rates, needing more bandwidth, so
  the data is spread across a range of frequencies: **spread spectrum**.
- **DSSS (direct sequence spread spectrum):** used in **2.4 GHz**. A small
  number of fixed, wide channels supporting complex **phase** modulation and
  somewhat scalable data rates. The channels are wide enough to augment the data
  by spreading it out, **making it more resilient to disruption**.
- **OFDM (orthogonal frequency division multiplexing):** used in **2.4, 5 and
  6 GHz**. A single 20 MHz channel carries data sent **in parallel** over
  multiple frequencies; the channel is divided into many **subcarriers** (also
  called subchannels or tones), and **both phase and amplitude are modulated
  with QAM (quadrature amplitude modulation)** to move the most data
  efficiently.

### 802.11 amendments
- The original standard was published in **1997** and is termed **Wi-Fi 0**, the
  root generation. Most amendments have been rolled into the overall 802.11
  standard and no longer stand alone, **but the industry still uses the task
  group names** — 802.11b was approved in 1999 and rolled up in 2007, and is
  still called 802.11b today.
- **802.11n (2009)** — theoretical max 600 Mbps; defined **high throughput (HT)**
  techniques usable on **either** 2.4 or 5 GHz.
- **802.11ac (2013)** — **very high throughput (VHT)**, **5 GHz only**. The
  866 Mbps per-stream figure is reachable **only** with every feature leveraged
  and favourable RF; there are around **320 different data rates**.
- **Everything up through 802.11ac assumes only one device can claim air time at
  a time.** **802.11ax (Wi-Fi 6, high efficiency / HE) changes that** — multiple
  devices may transmit during the same window of air time, which matters most in
  high-density areas. Roughly **four times** the ac data rates, via more complex
  and more sensitive modulation and coding.
- 802.11ax also uses **OFDMA** to schedule and control access, allocating air
  time as **resource units** usable by multiple devices simultaneously, and
  avoids neighbouring-BSS interference through better transmit power control and
  **BSS marking ("coloring")**.
- **Wi-Fi 6 = 802.11ax on 2.4 and 5 GHz. Wi-Fi 6E = 802.11ax on 6 GHz only.**
- 🚨 **The one concept to remember: an AP must support the same set of 802.11
  amendments its clients support.** If some clients are 802.11n-only and others
  are ac, the AP must support and be configured for both. Most amendments are
  backward compatible with previous ones **operating in the same band**.

### SISO, MIMO, and spatial streams
- Before 802.11n, devices had a single transmitter and single receiver — one
  **radio chain**, a **SISO (single-in, single-out)** system.
- **MIMO (multiple-input, multiple-output)** uses multiple antennas,
  transmitters and receivers, forming multiple radio chains.
- Devices are described as **T×R**: a **2×2** device has two transmitters and
  two receivers; a **2×3** has two transmitters and three receivers.
- **Spatial multiplexing** distributes data across two or more radio chains, all
  on the **same channel**, separated by **spatial diversity**. They avoid
  interfering because each chain has its own antenna: spaced apart, the signals
  arrive at the receiver's (also appropriately spaced) antennas **out of phase
  or at different amplitudes** — especially when they bounce off objects and
  travel slightly different paths.
- **Spatial streams** are independent data streams multiplexed over the radio
  chains; the receiver rebuilds them by reversing the multiplexing. This needs
  a good deal of DSP at **both** ends, and pays off in throughput — more streams,
  more data.
- Stream count is appended after a colon: **3×3:2** = three transmitters, three
  receivers, **two** spatial streams.
- 🚨 **The number of spatial streams is NOT tied to the number of radios.** It
  is tempting to assume one stream per transmitter/receiver, but streams are
  **distributed across** the chains; the possible count depends on the device's
  **processing capacity and transmitter feature set**, not its radio count.
- Mismatched support: the two devices **negotiate**, informing each other of
  their capabilities, and use the **lowest common** number of streams — though
  a transmitter can use an extra stream to **repeat information for redundancy**.

### Transmit beamforming and maximal-ratio combining
- **Transmit beamforming (TBF)** — a **transmitter-side** technique. Normally a
  single-chain transmitter shows no preference and every receiver is at the
  mercy of its own conditions. With MIMO, **the phase of the signal fed into
  each transmitting antenna is altered so the copies all arrive in phase at one
  specific receiver** — constructive interference, improving signal quality and
  SNR there. Devices not targeted receive the copies as-is, out of phase.
- TBF can use **explicit feedback** from the far-end device, letting the
  transmitter keep a table of devices and their phase adjustments and send
  focused transmissions to each dynamically.
- **Maximal-ratio combining (MRC)** — a **receiver-side** technique. A MIMO
  receiver gets multiple copies of the same signal on multiple antennas and
  chains. One copy may be better than the others, or better for a time and then
  worse. **MRC combines the copies to produce one signal representing the best
  version at any given moment**, yielding improved SNR and receiver sensitivity.
- **Remember them as a pair: TBF fixes the transmit side, MRC fixes the receive
  side.**

### Dynamic rate shifting
- Transmitter and receiver must use the **same** modulation method, and should
  use the **best data rate their current environment allows.** In a noisy
  environment with low SNR or low RSSI, a **lower** data rate is preferable.
- One way to fight FSPL is simply to **raise transmit power or antenna gain**,
  boosting EIRP and therefore RSSI at a distance. ⚠️ **This works for an
  isolated transmitter but causes interference problems when several
  transmitters share an area** — a very common self-inflicted design error.
- The robust approach is to cope with FSPL instead: as a client moves closer,
  **RSSI rises → SNR rises → more complex modulation and coding can be used →
  more data**. As it moves away, RSSI and SNR fall and more basic schemes are
  needed because of increased noise and retransmissions.
- **Dynamic rate shifting (DRS)** selects the scheme automatically with no
  manual intervention. Also called **link adaptation, adaptive modulation and
  coding (AMC), or rate adaptation**.
- 🚨 **DRS is not defined in the 802.11 standard.** Each manufacturer has its own
  approach, so **two devices in the same location will not necessarily choose
  the same scheme.**
- Each move outward into a larger concentric range ring shifts down to a reduced
  data rate to maintain data integrity at the edges; moving back toward the AP
  shifts the rates back up.

## Procedure

**Converting a power change to dB using the three laws (no calculator):**
1. Express the change from the reference to the value of interest as a chain of
   **×2 and ×10** operations. For 5 mW → 200 mW: **×2 → 10, ×2 → 20, ×10 → 200.**
2. Replace each **×2** with **+3 dB** and each **×10** with **+10 dB**
   (use −3 and −10 for ÷2 and ÷10).
3. Add them: **+3 +3 +10 = +16 dB**, so E = D + 16 dB.
4. The decomposition is not unique and any valid path gives the same answer —
   **×10 → 50, ×2 → 100, ×2 → 200** yields **+10 +3 +3 = +16 dB** as well.

**Calculating EIRP:**
1. Start with the **transmitter power level in dBm**.
2. **Subtract** the cable loss in dB between transmitter and antenna.
3. **Add** the antenna gain in dBi. *(If the gain is given in **dBd**, add 2.14
   first to convert it to dBi.)*
4. The result is the **EIRP in dBm** — e.g. **10 dBm − 5 dB + 8 dBi = 13 dBm**.
5. **Check it against the regulatory maximum EIRP** for the band and country
   before doing anything else with the number.

**Calculating received signal strength across the whole path:**
1. Begin with **transmit power in dBm** (100 mW = 20 dBm).
2. **Subtract** the transmit cable loss (−2 dB).
3. **Add** the transmit antenna gain (+4 dBi) — this point is the **EIRP**
   (20 − 2 + 4 = **22 dBm**).
4. **Subtract** the free space path loss between the antennas (−69 dB).
5. **Add** the receive antenna gain (+4 dBi).
6. **Subtract** the receive cable loss (−2 dB).
7. The result is the received signal in dBm: **20 − 2 + 4 − 69 + 4 − 2 =
   −45 dBm.** *(For perspective, a 69 dB Wi-Fi loss corresponds to roughly
   13–28 meters.)*
8. **Compare the result against the receiver's sensitivity level and the noise
   floor** — a number above sensitivity but only a few dB above the noise floor
   is still a bad link.

**Dynamic rate shifting as a client walks away from the AP:**
1. The client starts near the transmitter where RSSI and SNR are high, and uses
   the most complex scheme available — in the 2.4 GHz example, **OFDM 64-QAM 3/4
   at 54 Mbps.**
2. The user walks away; **RSSI falls, and with it SNR.**
3. The new RF conditions trigger a shift to a **less complex** modulation and
   coding scheme, resulting in a **lower data rate** but greater range.
4. Each further move outward crosses into a larger concentric ring and shifts
   down again, preserving data integrity at the outer reaches — ultimately down
   to **DSSS DBPSK at 1 Mbps**.
5. Moving back toward the AP reverses the process and the rates shift back up.
6. **The same walk in 5 GHz looks identical in shape, except every ring uses an
   OFDM scheme** corresponding to 802.11a/n/ac/ax.

## Reference Tables

**Frequency unit names**

| Unit | Abbreviation | Meaning |
|---|---|---|
| Hertz | Hz | Cycles per second |
| Kilohertz | kHz | 1,000 Hz |
| Megahertz | MHz | 1,000,000 Hz |
| Gigahertz | GHz | 1,000,000,000 Hz |

**Power changes and their corresponding dB values**

| Power change | dB value |
|---|---|
| = (equal) | 0 dB |
| × 2 | +3 dB |
| / 2 | −3 dB |
| × 10 | +10 dB |
| / 10 | −10 dB |

**Band characteristics compared**

| | 2.4 GHz | 5 GHz | 6 GHz |
|---|---|---|---|
| Channel spacing | **5 MHz** | 20 MHz | 20 MHz |
| Typical signal bandwidth | **22 MHz** | ~20 MHz | ~20 MHz |
| Adjacent channels overlap? | **Yes — signal is wider than 4 channels** | No | No |
| Wavelength | ≈ 4.92 in | ≈ 2.36 in | ≈ 1.97 in |
| Free space path loss | **Lowest** | Higher | **Highest** |
| Measured range to −67 dBm | **140 ft** | 80 ft | 50 ft |
| Spread spectrum used | DSSS and OFDM | OFDM | OFDM |

*The range figures come from an impromptu test carrying a receiver away from
co-located transmitters until RSSI hit −67 dBm. FSPL is the largest contributor
to the difference, but antenna size and receiver sensitivity differ between the
radios too.*

**Common 802.11 standard amendments**

| Standard | 2.4 GHz? | 5 GHz? | Data rates supported | Channel widths |
|---|---|---|---|---|
| **802.11b** (Wi-Fi 1) | Yes | No | 1, 2, 5.5, 11 Mbps | 22 MHz |
| **802.11a** (Wi-Fi 2) | No | Yes | 6, 9, 12, 18, 24, 36, 48, 54 Mbps | 20 MHz |
| **802.11g** (Wi-Fi 3) | Yes | No | 6, 9, 12, 18, 24, 36, 48, 54 Mbps | 22 MHz |
| **802.11n** (Wi-Fi 4) | Yes | Yes | Up to 150 Mbps per spatial stream, up to 4 streams | 20 or 40 MHz |
| **802.11ac** (Wi-Fi 5) | No | Yes | Up to 866 Mbps per spatial stream, up to 4 streams | 20, 40, 80, or 160 MHz |
| **802.11ax** (Wi-Fi 6) | Yes\* | Yes\* | Up to 1.2 Gbps per spatial stream, up to 8 streams | 20, 40, 80, or 160 MHz |

\* *802.11ax is designed to work on any band from 1 to 7 GHz, provided the band
is approved for use. Wi-Fi 6 covers 2.4 and 5 GHz; **Wi-Fi 6E is 6 GHz only**.*

**What a modulation scheme may alter**

| Attribute | Can be modulated? | Note |
|---|---|---|
| **Frequency** | Yes | **Only by varying slightly above or below the carrier frequency** |
| **Phase** | Yes | The basis of DSSS's complex phase modulation |
| **Amplitude** | Yes | Combined with phase in QAM |
| Anything else | **No** | These three are the only physical properties available |

**DSSS vs OFDM**

| | DSSS | OFDM |
|---|---|---|
| Bands | **2.4 GHz** | **2.4, 5 and 6 GHz** |
| Channel structure | Small number of fixed, wide channels | One 20 MHz channel divided into many **subcarriers** (subchannels/tones) |
| Transmission | Data spread across the wide channel | Data sent **in parallel** over multiple frequencies |
| Modulation | Complex **phase** modulation | **Phase and amplitude**, via **QAM** |
| Benefit | Spreading makes it **more resilient to disruption** | Moves the most data efficiently |

**Receiver measurements**

| Term | What it is | Watch out for |
|---|---|---|
| **RSSI** | 802.11-defined internal **1-byte relative value, 0–255** (0 weakest, 255 strongest) | 🚨 **No useful units, range varies by manufacturer, NOT standardized across receivers.** Usually presented already converted/scaled to dBm |
| **Sensitivity level** | Receiver threshold dividing intelligible from unintelligible signals | Above it the data will likely decode; below it, it will not |
| **Noise floor** | Average signal strength of all *other* signals on the same frequency | The value that most often changes when "nothing changed" |
| **SNR** | **Signal minus noise floor, in dB. Higher is better** | A strong signal with a risen noise floor is still a bad link |

**Typical power ranges**

| Point in the path | Range in mW | Range in dBm |
|---|---|---|
| EIRP leaving the transmit antenna | 100 mW – 1 mW | **+20 dBm – 0 dBm** |
| Arriving at the receiver | 1 mW – fractions approaching 0 | **0 dBm – about −100 dBm** |

## Config Patterns

⚠️ **Chapter 17 is conceptual and contains no device configuration.** The math
below is the chapter's real content. The IOS-XE C9800 commands that follow are
the *applied* form of these concepts and are included for usefulness — **they
are not from the chapter, and WLC syntax varies by platform and release, so
verify against your controller's own configuration guide before using them.**

```text
! ===== The formulas this chapter is actually testing =====
dB          = 10 x log10(P2 / P1)              ! P2 = source of interest, P1 = reference
dBm         = dB referenced to a fixed 1 mW
dBi -> from dBd:  dBi = dBd + 2.14             ! dipole reference -> isotropic reference

EIRP        = Tx Power - Tx Cable + Tx Antenna         ! result in dBm; legally capped
Rx Signal   = Tx Power - Tx Cable + Tx Antenna
                        - Free Space
                        + Rx Antenna - Rx Cable        ! the full link budget
SNR         = Signal (dBm) - Noise Floor (dBm)         ! result in dB; higher is better
FSPL (dB)   = 20 log10(d) + 20 log10(f) + 32.44        ! d in km, f in MHz - not on the exam

! Worked example - EIRP
!   10 dBm transmitter - 5 dB cable + 8 dBi antenna = 13 dBm EIRP
! Worked example - full path
!   20 dBm - 2 dB + 4 dBi - 69 dB + 4 dBi - 2 dB = -45 dBm at the receiver
! Worked example - dB laws, 5 mW -> 200 mW
!   x2, x2, x10  =>  +3 +3 +10  =>  +16 dB
```

```ios-xe
! ===== Applied: Catalyst 9800 WLC — NOT from Chapter 17, verify per release =====
! Lock 2.4 GHz to 20 MHz and to the three non-overlapping channels
ap dot11 24ghz rrm channel dca chan-width 20
ap dot11 24ghz rrm channel dca remove 2
ap dot11 24ghz rrm channel dca remove 3
!  ... leaving only 1, 6 and 11 in the DCA list

! An RF profile is where coverage/SNR thresholds actually live
wireless rf-profile RF-VOICE-24
 coverage data  -67
 coverage voice -67
 coverage level global 3
 channel width 20

! Verification
show ap dot11 24ghz summary            ! per-AP channel, tx power level, channel width
show ap auto-rf dot11 24ghz            ! RRM view: noise floor, channel utilisation, neighbours
show ap dot11 24ghz channel            ! DCA channel list actually in use
show wireless client summary           ! associated clients
show wireless client mac-address <mac> detail   ! per-client RSSI, SNR, data rate, spatial streams
show ap rf-profile summary
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Design the cell edge to **−67 dBm RSSI or better** for voice-grade coverage | Below this, modulation shifts down under DRS and retransmissions climb; −67 dBm is the figure Cisco's own site-survey guidance builds the coverage goal around | Data-only or scanner/IoT coverage where a lower edge is acceptable and deliberately chosen; outdoor or warehouse designs with different goals | [Cisco VoWLAN Site Survey and RF Design Validation](https://www.cisco.com/c/en/us/td/docs/wireless/technology/vowlan/troubleshooting/vowlan_troubleshoot/8_Site_Survey_RF_Design_Valid.html) |
| Require **SNR of at least 25 dB**, assuming a noise floor no worse than **−92 dBm** | **A good RSSI with a risen noise floor is still an unusable link** — SNR, not signal strength, is what determines whether data decodes | A tolerant, low-rate application; an environment with a known and accepted higher noise floor, documented as a risk | [Cisco VoWLAN Site Survey and RF Design Validation](https://www.cisco.com/c/en/us/td/docs/wireless/technology/vowlan/troubleshooting/vowlan_troubleshoot/8_Site_Survey_RF_Design_Valid.html) |
| Plan **20% cell overlap at 2.4 GHz and 15–20% at 5 GHz**, between APs on **different** channels | Too little overlap creates coverage holes at roam boundaries; overlap on the *same* channel creates co-channel interference instead of redundancy | Deliberately sparse coverage for cost reasons in low-value areas, accepted as a coverage gap | [Cisco VoWLAN Site Survey and RF Design Validation](https://www.cisco.com/c/en/us/td/docs/wireless/technology/vowlan/troubleshooting/vowlan_troubleshoot/8_Site_Survey_RF_Design_Valid.html) |
| Use **only channels 1, 6 and 11** in 2.4 GHz, locked to **20 MHz** | These are the only non-overlapping channels in the band; **the 22 MHz signal is wider than four 5 MHz channels**, so any other plan guarantees adjacent-channel interference | Regulatory domains permitting channels 12–14 may support a different non-overlapping set; a site with only one or two APs where reuse is moot | [Cisco Wireless High Client Density Design Guide](https://www.cisco.com/c/en/us/td/docs/wireless/controller/technotes/8-7/b_wireless_high_client_density_design_guide.html) |
| Keep **channel utilisation under 50%**, **retransmissions under 20%**, **packet loss under 1%**, **jitter under 100 ms** | These are the companion figures to the RSSI/SNR targets; a link can meet the signal numbers and still fail on air-time contention | Best-effort data networks where voice-grade thresholds were never the goal | [Cisco VoWLAN Site Survey and RF Design Validation](https://www.cisco.com/c/en/us/td/docs/wireless/technology/vowlan/troubleshooting/vowlan_troubleshoot/8_Site_Survey_RF_Design_Valid.html) |
| **Do not solve coverage problems by raising transmit power** in a multi-AP environment | Raising EIRP boosts RSSI at distance for **one** transmitter but **causes interference problems when several transmitters are in an area** — and it cannot fix the client's own weaker transmit power, creating an asymmetric link | A genuinely isolated AP with no neighbours, where the trade does not exist | ENCOR OCG Ch.17, "Maximizing the AP–Client Throughput" |
| **Verify EIRP against the regulatory maximum** for the band and country before deployment | **EIRP is regulated by government agencies in most countries**; a system may not radiate above the allowable maximum. This is a legal limit, not a tuning knob | None — this is a compliance boundary, not a design preference | ENCOR OCG Ch.17, "Measuring Power Changes Along the Signal Path" |

*A deviation from this table is a question for the network's operator — "is
this intentional here?" — never automatically a finding.*

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show wireless client mac-address <mac> detail` | **Per-client RSSI and SNR** — the two numbers that decide whether a complaint is real. Also shows current data rate, spatial streams, and the 802.11 protocol in use |
| `show ap auto-rf dot11 24ghz` / `... 5ghz` | **Noise floor**, channel utilisation, interference, and the AP's RRM neighbour list. **This is where you catch a risen noise floor** — the cause of an SNR collapse when signal strength looks fine |
| `show ap dot11 24ghz summary` | Per-AP channel assignment, transmit power level, and channel width. **Confirms whether 2.4 GHz is actually locked to 20 MHz and to 1/6/11** |
| `show ap dot11 24ghz channel` | The DCA channel list actually in use — catches channels 2–5 and 7–10 left enabled |
| `show ap dot11 5ghz summary` | Channel width in 5 GHz — a 40/80/160 MHz width in a dense deployment reduces the number of non-overlapping channels available |
| `show ap rf-profile summary` / `show ap rf-profile detailed <name>` | Which RF profile a group is using and its coverage/SNR thresholds — the configured intent, versus what the other commands show as reality |
| `show wireless client summary` | Which AP each client is on, and its protocol — catches clients stuck on a distant AP at a low rate |
| A **spectrum analyzer** or CleanAir-capable AP | **Non-802.11 interference does not appear in any of the above** — a microwave, video bridge, or Bluetooth source raises the noise floor invisibly to Wi-Fi-only tooling |

## Intent Questions
- **Coverage intent:** what is this WLAN supposed to *carry* — best-effort data,
  voice, or location/real-time services? **The answer sets the cell-edge RSSI
  and SNR targets, and every other number follows from it.** A design that is
  fine for barcode scanners is a failure for voice.
- **Band intent:** which bands are clients expected to use, and is 2.4 GHz meant
  to be primary coverage or a legacy fallback? 2.4 GHz has the greatest range
  but only three usable channels.
- **Density intent:** is this designed for coverage (few APs, wide cells) or
  capacity (many APs, small cells)? **The two designs look opposite** — a
  capacity design deliberately *lowers* transmit power, which reads as "wrong"
  to anyone expecting a coverage design.
- **Client intent:** what is the oldest 802.11 amendment that must still be
  supported? **The AP must support the same amendments its clients do**, and one
  legacy client family can constrain the whole cell.

## Troubleshooting Checklist
0. **State intent vs. observed:** answer the Intent Questions above, then write
   the one-line symptom ("voice should hold at −67 dBm at the cell edge, but
   clients drop to −78 dBm in the east corridor"). **Do this before opening any
   tool** — "the Wi-Fi is slow" is not a symptom.
1. **Get RSSI *and* SNR for the complaining client**, not just signal strength.
   `show wireless client mac-address <mac> detail`. **A good RSSI with a low SNR
   is a noise problem, not a coverage problem, and the fixes are opposite.**
2. **Check the noise floor** with `show ap auto-rf`. If SNR collapsed while the
   signal held steady, **the noise floor moved** — look for a new interferer
   rather than re-surveying coverage.
3. **Check for non-802.11 interference.** A microwave oven, video bridge, or
   Bluetooth source raises the noise floor and **will not appear** in any
   Wi-Fi-only output. This needs a spectrum analyzer or CleanAir.
4. **Check channel utilisation and retransmissions.** A link can meet every
   signal target and still be unusable because the air is busy — target under
   50% utilisation and under 20% retransmissions.
5. **Check the channel plan in 2.4 GHz.** Anything other than 1/6/11 at 20 MHz
   means adjacent-channel overlap by construction — **the 22 MHz signal is wider
   than four 5 MHz channels.** Verify with `show ap dot11 24ghz channel`.
6. **Check channel width in 5 GHz.** A wide 40/80/160 MHz channel raises peak
   rates but reduces the number of non-overlapping channels, which in a dense
   deployment trades throughput for co-channel interference.
7. **Check co-channel interference and cell overlap.** Overlap between APs on
   the *same* channel is not redundancy — it is contention. Target 20% overlap
   at 2.4 GHz and 15–20% at 5 GHz **between APs on different channels**.
8. **Check transmit power symmetry.** An AP turned up to overcome a coverage
   hole can hear a client that cannot hear it back — the client's transmit power
   is fixed and much lower. **This asymmetry looks like a one-way failure.**
9. **Check the band and the range expectation.** If clients fail at distance on
   5 or 6 GHz but work on 2.4 GHz, that is **free space path loss behaving
   exactly as designed** — roughly 140 / 80 / 50 ft to −67 dBm — not a fault.
10. **Check amendment and capability mismatch.** Confirm the AP supports the
    amendments its clients need, and check spatial-stream negotiation — a
    3×3:2 AP and a 1×1 client will settle on the **lowest common** number.
11. **Check the link budget arithmetic** if this is a new install or a
    point-to-point link: Tx power − Tx cable + Tx antenna − FSPL + Rx antenna −
    Rx cable. **Verify the EIRP is inside the regulatory maximum**, and confirm
    the antenna gain units — **dBd is not dBi; add 2.14.**
12. **Only then consider raising power**, and only if the AP is genuinely
    isolated. In a multi-AP area, raising power moves the problem rather than
    fixing it.

## Common Pitfalls
- **Reading signal strength alone and calling it good.** SNR is what decides
  whether data decodes. A −54 dBm signal is excellent against a −90 dBm noise
  floor (36 dB) and marginal against a −65 dBm one (11 dB) — **the signal never
  changed.**
- **Treating RSSI as a standardized, comparable number.** 802.11 defines it as an
  internal **relative** 0–255 value with **no units**, and **the scale varies
  between manufacturers**. Comparing a raw RSSI from two different client types
  is comparing nothing.
- **Assuming adjacent 2.4 GHz channel numbers don't overlap.** Channels are
  5 MHz apart and the signal is 22 MHz wide — **wider than four channels.** Only
  1, 6 and 11 are non-overlapping.
- **Assuming the standardized channel numbering guarantees non-overlap in any
  band.** Channels get numbered for one technology, and later technologies reuse
  the band with wider signals.
- **Confusing dBd with dBi.** A dipole has 2.14 dBi of gain; **add 2.14 to a dBd
  figure before using it in an EIRP calculation.** This is the one unit that
  cannot be combined directly.
- **Trying to express antenna gain in dBm.** An antenna produces no absolute
  power of its own — gain is only ever a **comparison**, in dB.
- **Forgetting that EIRP is legally capped.** It is regulated by government
  agencies in most countries; the maximum is a compliance boundary, not a
  tuning ceiling you can raise.
- **Assuming indoor free space path loss is negligible** because distances are
  short and the formula takes kilometers. **Even at 1 meter, FSPL costs about
  46 dB.**
- **Expecting equal range across bands.** FSPL rises with frequency, so
  2.4 > 5 > 6 GHz for range at equal transmit power — measured at roughly
  140 / 80 / 50 ft to −67 dBm.
- **Raising transmit power to fix coverage in a multi-AP environment.** It
  causes interference between transmitters and creates an asymmetric link the
  client cannot hold up its end of.
- **Assuming spatial streams equal radio count.** A 3×3:2 device has three
  transmitters, three receivers and **two** streams; stream count depends on
  processing capacity and feature set, not radio count.
- **Assuming two devices in the same spot will pick the same data rate.**
  **DRS is not defined in the 802.11 standard** — every manufacturer implements
  it differently.
- **Forgetting the AP must support the amendments its clients use.** Mixed
  legacy and modern clients require the AP to support and be configured for
  both; backward compatibility only applies **within the same band**.
- **Mixing up TBF and MRC.** **Transmit beamforming is transmitter-side** (phase
  the copies so they arrive in phase at one receiver); **maximal-ratio combining
  is receiver-side** (combine received copies into the best version).
