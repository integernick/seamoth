# AGILE REEF-SURVEY GLIDER

**Technical Requirements & Architecture Specification**

*Codename: VBC GLIDER v1*

Middle-Aged Engineering — March 2026 — v0.1

Nick Bondarev · Daniil Izmaylov · Dmitry Kulikov · Kristina · Pavel Vladimirovich

---

## 1. Mission profile

Autonomous reef survey in warm shallow water. Silent operation (no thrusters). Agile maneuvering around coral structures.

| Parameter | Value |
|---|---|
| **Primary mission** | Coral reef monitoring, photogrammetry, tube/structure inspection |
| **Operating depth** | 5–20 m (rated to 30 m) |
| **Environment** | Arabian Gulf, 25–35°C, ~40 ppt salinity |
| **Endurance target** | 4–8 hours active survey (48 h theoretical max) |
| **Autonomy** | Full: waypoint nav, obstacle avoidance, vision processing |
| **Recovery** | GPS surface fix, drop-weight emergency ascent |

---

## 2. Vehicle overview

### 2.1 Form factor

| Parameter | Value |
|---|---|
| **Hull** | BlueRobotics 4″ WTE, 114.3 mm OD, ~100 mm ID |
| **Body length** | ~700 mm (hull + nose + tail) |
| **Total wingspan** | ~530 mm (body + 2 × 161 mm semi-span) |
| **Mass (target)** | 8–10 kg, neutrally trimmed |
| **Propulsion** | Oil-hydraulic buoyancy engine, 4 independent external bladders (VBC) |
| **Control surfaces** | None — all attitude control via vectored buoyancy |

### 2.2 Estimated performance

| Parameter | Value |
|---|---|
| **Glide angle** | 14–18° from horizontal (L/D 3.5–4.5) |
| **Forward speed** | 0.5–0.8 m/s (1.8–2.9 km/h) |
| **Turning radius** | <5 m via differential bladder roll + banked turn |
| **Pitch transition** | 3–7 s (single pump), 1.5–3.5 s (dual pump) |
| **Acoustic signature** | Near-zero (gear pump only) |
| **Energy cost** | 1.3–3.1 Wh/km |

*Reference vehicles: ROUGHIE (Purdue) for agility benchmark, ReefGlider (WHOI) for vectored buoyancy architecture.*

---

## 3. Buoyancy engine

Single pump distributes mineral oil between internal reservoir and 4 external bladders via solenoid valves.

### 3.1 Hydraulic parameters

| Parameter | Value |
|---|---|
| **Working fluid** | Mineral hydraulic oil, ISO VG 22 (Shell Tellus S2 V 22) |
| **Total variable volume** | ±120–160 mL (30–40 mL per bladder) |
| **Operating pressure** | 1–4 bar gauge |
| **Net buoyancy for 0.6 m/s** | ~80 mL (~0.8 N) |

### 3.2 VBC mapping

| Motion | Fwd bladders | Aft bladders | Effect |
|---|---|---|---|
| Dive | Inflate both | Deflate both | COB shifts fwd of COM |
| Climb | Deflate both | Inflate both | COB shifts aft of COM |
| Roll right | Inflate port | Inflate port | COB shifts port → bank right |
| Roll left | Inflate stbd | Inflate stbd | COB shifts stbd → bank left |
| Ascend | Inflate all | — | Net positive buoyancy |
| Descend | Deflate all | — | Net negative buoyancy |

---

## 4. Hydrodynamics

### 4.1 Wing specification

| Parameter | Value |
|---|---|
| **Airfoil** | SD7003 (Selig, UIUC) — Re 40k–80k optimized |
| **Total wing area S** | 300 cm² (0.030 m²) |
| **Aspect ratio AR** | 6 |
| **Chord c** | 71 mm |
| **Semi-span** | 161 mm each side |
| **Dihedral** | 3–5° |
| **Mounting** | Clamshell clamp + streamlined fairing, 55–65% body length from nose |

### 4.2 Drag budget at 0.75 m/s

| Component | Cd / basis | Drag (N) | Share |
|---|---|---|---|
| Hull parasitic | Cd=0.11, frontal 81.7 cm² | 0.26 N | 38% |
| Wing profile | Cd0=0.025, S=300 cm² | 0.22 N | 32% |
| Wing induced | CL=0.5, AR=6, e=0.85 | 0.13 N | 19% |
| Appendages | Tail, antenna, fairing | 0.07 N | 11% |
| **TOTAL** | — | **0.68 N** | 100% |

*Predicted total lift at CL=0.5: 4.32 N. L/D ≈ 4.0–4.5. Gulf water at 28–32°C raises Re by ~25% vs cold-water assumptions.*

### 4.3 Calculation summary (Graver 2005)

*Following Graver, J.G. (2005) "Underwater Gliders: Dynamics, Control and Design", Princeton University, Chapters 4 and 7. Full calculations available in the companion Jupyter notebook (`vbc_glider_calcs.ipynb`).*

#### Hydrodynamic coefficients (Graver §4.1.2)

Drag polar: CD = CD0 + CD,α · α². Lift: CL = CL0 + CL,α · α. Lumped K parameters: Ki = ½ρ · S · Ci.

| Parameter | Value |
|---|---|
| **CD0 (total, ref S)** | 0.063 (hull 0.030 + wing 0.025 + appendages 0.008) |
| **CD,α** | 1.064 rad⁻² |
| **CL,α (total incl. hull)** | 5.1 rad⁻¹ |
| **CL0 (SD7003 camber)** | 0.15 |
| **KD0** | 0.967 N·s²/m² |
| **KD** | 16.35 N·s²/m²·rad⁻² |
| **KL0** | 2.302 N·s²/m² |
| **KL** | 78.35 N·s²/m²·rad⁻¹ |

#### Reynolds number

Gulf water at 28°C: ν = 0.88 × 10⁻⁶ m²/s (~25% lower than 15°C standard).

| Glide speed | Re (wing chord) | Re (hull) | Regime |
|---|---|---|---|
| 0.3 m/s | 24,200 | 238,600 | Low-Re, LSB risk |
| 0.5 m/s | 40,300 | 397,700 | SD7003 effective zone |
| 0.7 m/s | 56,500 | 556,800 | Optimal for SD7003 |
| 1.0 m/s | 80,700 | 795,500 | Approaching turbulent transition |

#### Steady glide equilibria (Graver Eq. 4.18–4.23, 7.4)

tan(ξ) = D/L. Equilibrium α from Eq. 4.21. Speed V = √(m0·g / [(-sinξ)(KD0 + KD·α²) + (cosξ)(KL0 + KL·α)]).

| ξ [°] | αd [°] | L/D | V @80 mL | V @160 mL | Vh @160 mL |
|---|---|---|---|---|---|
| 10 | 2.1 | 5.7 | 0.47 m/s | 0.67 m/s | 0.66 m/s |
| 15 | 3.2 | 3.7 | 0.54 m/s | 0.77 m/s | 0.74 m/s |
| 20 | 4.4 | 2.7 | 0.58 m/s | 0.82 m/s | 0.77 m/s |
| 25 | 5.6 | 2.1 | 0.60 m/s | 0.85 m/s | 0.77 m/s |

*Optimal horizontal speed peaks at ξ ≈ 15°. Ballast fraction β = m0/mv = 1.88% (cf. Slocum 1.5%, Seaglider 0.8%). Higher β intentionally favors speed.*

#### Energy per cycle (Graver §7.2.1)

| Parameter | Value |
|---|---|
| **Volume change per full reversal** | 320 mL (2 × 160 mL) |
| **Avg. pressure (sawtooth to 15 m)** | ~1.5 bar |
| **Mechanical energy / cycle** | 48 J |
| **Electrical energy / cycle** | 107 J = 0.030 Wh (at η = 45%) |
| **Pump run time / cycle** | ~38 s |
| **Cycles from 116 Wh (40% to pumping)** | ~1,550 |

---

## 5. Compute architecture

Heterogeneous dual-processor: always-on dual-core MCU for flight control + estimation, duty-cycled Jetson for vision + path planning. Connected via UART + FDCAN.

### 5.1 Flight controller — STM32H755 (always-on)

| Parameter | Value |
|---|---|
| **MCU** | STM32H755ZIT6 (Cortex-M7 @ 480 MHz + Cortex-M4 @ 240 MHz) |
| **RTOS** | Zephyr (both cores, AMP configuration) |
| **FPU** | Double-precision (M7), single-precision (M4) |
| **RAM** | 1 MB (864 KB M7, 288 KB M4, shared region configurable) |
| **Board** | Custom PCB (prototype on NUCLEO-H755ZI-Q) |

### 5.2 Core allocation

| Core | Responsibilities | Rate | Notes |
|---|---|---|---|
| M7 | IEKF state estimation (quaternion, 9+ states) | 1 kHz | Double-precision FPU, DMA sensor reads |
| M7 | Short-horizon attitude MPC (4 bladder outputs) | 50–100 Hz | Runs after IEKF update |
| M4 | Sensor polling: IMU (SPI), Bar30 (I²C), Ping2 (UART) | 1 kHz | DMA to shared RAM |
| M4 | Actuator output: pump PWM, 4× valve GPIO, LED strobe | 100 Hz | Deterministic, no jitter |
| M4 | Housekeeping: battery ADC, leak, temp, SD log | 10 Hz | Low-priority thread |
| M4 | Comms bridge: UART to Jetson + GPS | Async | Message routing |
| M4 | **SAFETY SUPERVISOR (see §5.4)** | 100 Hz | Independent watchdog loop |

### 5.3 Vision / autonomy — Jetson Orin (duty-cycled)

| Parameter | Value |
|---|---|
| **SOM** | NVIDIA Jetson Orin Nano 8 GB (40 TOPS) or Orin NX 8 GB (70 TOPS) |
| **Carrier** | Auvidea JNX120S (24.7 × 75 mm) or Connect Tech Hadron |
| **Power (active / SC7 / off)** | 7–15 W / 0.3 W / 0 W (MOSFET-gated by M4) |
| **Camera** | MIPI CSI-2 native |
| **OS** | JetPack 6.x (Ubuntu 22.04 + CUDA + TensorRT) |
| **Duty cycle** | ~8% nominal, higher during active survey |

### 5.4 Dual-core fault tolerance

The H755 dual-core architecture enables hardware-level fault isolation. M4 acts as an independent safety supervisor that can detect M7 failure and execute emergency recovery without any M7 involvement.

#### Architecture: M4 as safety supervisor

| Mechanism | Implementation | Timing |
|---|---|---|
| M7 heartbeat | M7 increments a counter in shared SRAM every 10 ms. M4 checks at 100 Hz. | Timeout: 100 ms |
| M7 targeted reset | M4 asserts RCC_AHB3RSTR.CPURST (resets M7 core only, M4 unaffected) | Recovery: <500 ms |
| M7 recovery check | After reset, M4 waits for M7 heartbeat to resume within 2 s | Timeout: 2 s |
| Emergency surface | If M7 fails to recover: M4 opens all 4 valves (drain bladders → positive buoyancy) | Immediate |
| Jetson kill | M4 de-asserts Jetson MOSFET gate (saves power for GPS beacon) | Immediate |
| GPS beacon mode | M4 activates GPS + LoRa at surface, transmits position every 60 s | Continuous |
| Drop weight (last resort) | If depth keeps increasing 30 s after valve-open: fire nichrome burn wire | 30 s deadline |

#### Fail-safe state table

| Failure mode | Detection | M4 response | Expected outcome |
|---|---|---|---|
| M7 hardfault / segfault | Heartbeat timeout >100 ms | Reset M7 → if no recovery, surface | Glider surfaces, GPS beacon |
| Jetson hang | Jetson heartbeat timeout >5 s | Kill Jetson MOSFET, M7 holds depth or surfaces | Continues on M7 only |
| Leak detected | SOS sensor EXTI interrupt | ALL STOP: valves open, pump off, surface, drop weight arm | Immediate ascent |
| Battery critical (<10%) | ADC threshold on M4 | Surface, disable Jetson, GPS beacon | Preserves power for recovery |
| Both M7 + Jetson fail | M4 alone | Valves open (positive buoyancy), drop weight after 30 s | Passive surfacing |
| M4 fails | M7 IWDG (hardware watchdog) | H755 full system reset (both cores reboot) | Restarts from safe state |

*Key design principle: M4 safety supervisor code is minimal, heavily tested, and never updated during missions. It runs in a protected memory region with MPU isolation. The M7 IEKF/MPC code can be iterated freely without risking the safety layer.*

#### Hardware watchdog stack

| Parameter | Value |
|---|---|
| **IWDG1 (M7)** | Independent watchdog, LSI-clocked, resets M7 if not kicked within 200 ms |
| **IWDG2 (M4)** | Independent watchdog, resets entire H755 if M4 safety loop stalls |
| **WWDG (M7)** | Window watchdog for IEKF timing validation (must kick within 8–12 ms window) |
| **Shared SRAM heartbeat** | Software heartbeat M7→M4 via HSEM-protected counter |
| **FDCAN bus** | Secondary heartbeat channel (M7 broadcasts state at 10 Hz on CAN) |

### 5.5 Control hierarchy

| Layer | Runs on | Function | Rate |
|---|---|---|---|
| Estimation | H755 M7 | IEKF: attitude + position + velocity | 1 kHz |
| Inner control | H755 M7 | Attitude MPC: targets → 4 valve commands | 50–100 Hz |
| Outer guidance | Jetson | Path MPC: waypoints + obstacles → attitude targets | 1–5 Hz |
| Perception | Jetson | Camera reef classification, sonar obstacle map | 1–5 FPS |
| Mission | Jetson | Survey pattern, abort logic, surface scheduling | Event |
| Emergency | H755 M4 | Independent of M7 + Jetson (see §5.4) | Always-on |

### 5.6 Inter-processor communication

| Parameter | Value |
|---|---|
| **M7 ↔ M4** | Shared SRAM + HSEM (hardware semaphore), zero-copy |
| **H755 ↔ Jetson (primary)** | UART, 921600 baud, binary protocol |
| **H755 ↔ Jetson (secondary)** | FDCAN, 1 Mbit/s |
| **Jetson power control** | M4 GPIO → high-side MOSFET, GPIO wake pin |
| **Fail-safe** | Jetson heartbeat lost >5 s: M7 enters hold-depth or surface mode |

---

## 6. Custom flight controller PCB

Target board size: 90 × 60 mm. Fits flat inside 4″ hull (~95 mm usable width).

### 6.1 Required interfaces

| Interface | Connected to | H755 peripheral | Notes |
|---|---|---|---|
| SPI (high-speed) | IMU (ICM-42688-P onboard) | SPI1 (M4) | Vibration-isolated footprint |
| I²C #1 | Bar30 + ABP2 + mag | I2C1 (M4) | External connector |
| UART #1 | Jetson SBC | USART1 (M4) | 921600 baud |
| UART #2 | Ping2 altimeter | USART2 (M4) | 115200, TTL |
| UART #3 | GPS (NEO-M9N) | USART3 (M4) | 38400 |
| UART #4 | Iridium (future) | UART5 (M4) | 19200 |
| FDCAN | Jetson (redundant) | FDCAN1 | 1 Mbit/s, CAN-FD |
| PWM #1 | Pump BLDC via ESC | TIM1 CH1 (M4) | Variable speed |
| GPIO × 4 | Solenoid valves | M4 GPIO | Via MOSFET driver |
| GPIO × 2 | LED strobe triggers | M4 GPIO | Camera sync |
| GPIO | Jetson power MOSFET | M4 GPIO | High-side switch |
| GPIO | Drop-weight nichrome | M4 GPIO | SW-armed only |
| GPIO (EXTI) | Leak sensor (SOS) | M4 GPIO | Interrupt-driven |
| ADC | Battery V divider + NTC thermistor | ADC1 (M4) | 12-bit |
| I²C | INA219 battery current sense | I2C1 | Coulomb counting |
| SPI #2 | MicroSD card | SPI2 (M4) | Data logging |
| USB-C | Debug / firmware flash | USB OTG FS | DFU + Zephyr shell |
| SWD | Debug header | Cortex debug | 10-pin ARM standard |

### 6.2 Onboard components

| Component | Part | Notes |
|---|---|---|
| MCU | STM32H755ZIT6 (LQFP-144) | Dual-core, 2 MB flash, 1 MB RAM |
| IMU | ICM-42688-P (soldered) | 6-DOF, 32 kHz ODR |
| Magnetometer | LIS3MDL or MMC5983MA | Away from motor traces |
| Barometer | BMP390 (backup altitude ref) | I²C |
| Power reg | 5V (TPS54331) + 3.3V (LDO) | Separate analog/digital domains |
| MOSFET drivers | 4-ch valve driver + Jetson switch | Flyback diodes for solenoids |
| CAN transceiver | TCAN332G | CAN-FD compatible |
| Connectors | JST-GH (Pixhawk-compatible pinout) | Keyed, locking |

*PCB stack-up: 4-layer (signal/GND/power/signal). Ground plane unbroken under IMU.*

---

## 7. Sensors

| Sensor | Model | Interface | Avg power |
|---|---|---|---|
| Depth | [BlueRobotics Bar30](https://bluerobotics.com/store/sensors-cameras/sensors/bar30-sensor-pcb-r1/) | I²C | <0.01 W |
| IMU | ICM-42688-P (onboard PCB) | SPI | ~0.005 W |
| Magnetometer | LIS3MDL (onboard PCB) | I²C | ~0.003 W |
| Altimeter | [BlueRobotics Ping2](https://bluerobotics.com/store/sonars/echosounders/ping-sonar-r2-rp/) | UART | ~0.1 W |
| Line pressure | Honeywell ABP2 | I²C | <0.01 W |
| Camera | [RPi Camera Module 3 Wide NoIR](https://www.raspberrypi.com/products/camera-module-3/) | MIPI CSI-2 | ~0.04 W |
| Lighting | [2× BlueRobotics Lumen](https://bluerobotics.com/store/thrusters/lights/lumen-r2-rp/) | PWM strobe | ~0.6 W |
| GPS | [u-blox NEO-M9N](https://www.u-blox.com/en/product/neo-m9n-module) | UART | ~0.04 W |
| Leak | [BlueRobotics SOS](https://bluerobotics.com/store/sensors-cameras/leak-sensor/sos-leak-sensor/) | GPIO | <0.01 W |

*v2 additions: Water Linked DVL A50 (dead reckoning), Tritech Micron Gemini (FLS), RockBLOCK 9603N (Iridium).*

---

## 8. Power budget

### 8.1 Battery

| Parameter | Value |
|---|---|
| **Configuration** | 3S3P Samsung INR18650-35E |
| **Capacity** | 11.1 V / 10.5 Ah / 116 Wh |
| **Mass** | ~425 g |
| **BMS** | Daly 3S 20A |

### 8.2 Power breakdown

| Subsystem | Active (W) | Duty | Average (W) |
|---|---|---|---|
| H755 flight controller | 0.35 | 100% | 0.35 |
| Sensors (all) | 0.08 | 100% | 0.08 |
| Ping2 altimeter | 0.5 | 20% | 0.10 |
| Jetson Orin (7W mode) | 7.0 | 8% | 0.56 |
| Jetson SC7 standby | 0.3 | 92% | 0.28 |
| Camera | 0.5 | 8% | 0.04 |
| LED strobes (2×) | 30 | 0.2% | 0.06 |
| GPS + comms (surface) | 2.0 | 2% | 0.04 |
| Solenoid valves (4×) | 3.0 | 10% | 0.30 |
| DC-DC losses (~10%) | — | — | 0.19 |
| Buoyancy pump | 8.0 | 5% | 0.40 |
| **TOTAL** | — | — | **2.40** |

**Endurance:** 116 Wh ÷ 2.40 W ≈ 48 h continuous. Active survey (30% Jetson duty): ~28 h.

---

## 9. Bill of materials

### 9.1 Buoyancy engine

| Part | Model | Qty | Est. cost |
|---|---|---|---|
| Oil pump | [TCS MGD1000S-PK-V](https://micropumps.co.uk/micro-gear-pumps-mg-series/) | 1 | £200–400 |
| Solenoid valves | [Gems A2016-C203 Viton 12V](https://www.amazon.com/Gems-Sensors-A2016-C203-Stainless-Solenoid/dp/B009NAQRYG) | 4 | $320–480 |
| Compensator | [Macduff Robotics 275 cc](https://macduff-robotics.com/compensators.html) | 1 | $100–300 |
| Bladders | NBR rubber, 25 mm OD, 40 Shore A | 4 | $20–50 |
| Bladder mesh | Nylon braided sleeve 25–40 mm | 4 | $10–20 |
| Oil | [Shell Tellus S2 V 22](https://www.shell.com/business-customers/lubricants-for-business/shell-tellus) | 1 L | $10–30 |
| Tubing | PTFE 3 mm ID / 5 mm OD | 2 m | $15–30 |
| Check valve | 316 SS spring type | 1 | $15–40 |
| Fill/bleed valve | 316 SS ball valve 1/8″ NPT | 1 | $10–20 |
| Oil bulkheads | [Swagelok SS-200-61](https://products.swagelok.com/en/c/straights/p/SS-200-61) | 4 | $100–140 |

### 9.2 Electronics & compute

| Part | Model | Qty | Est. cost |
|---|---|---|---|
| Flight controller | Custom PCB (STM32H755ZIT6) | 1 | $80–150 |
| Dev board | [NUCLEO-H755ZI-Q](https://www.st.com/en/evaluation-tools/nucleo-h755zi-q.html) | 1 | $30 |
| Jetson SOM | [Orin Nano 8 GB / Orin NX 8 GB](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/) | 1 | Owned |
| Jetson carrier | [Auvidea JNX120S](https://auvidea.eu/product-category/jetson/orin-nx-nano/) | 1 | $150–350 |
| Depth sensor | [BlueRobotics Bar30](https://bluerobotics.com/store/sensors-cameras/sensors/bar30-sensor-pcb-r1/) | 1 | $90 |
| Line pressure | Honeywell ABP2 board-mount | 1 | $15–25 |
| Altimeter | [BlueRobotics Ping2](https://bluerobotics.com/store/sonars/echosounders/ping-sonar-r2-rp/) | 1 | $430 |
| Camera | [RPi Camera 3 Wide NoIR](https://www.raspberrypi.com/products/camera-module-3/) | 1 | $35 |
| Lights | [BlueRobotics Lumen](https://bluerobotics.com/store/thrusters/lights/lumen-r2-rp/) | 2 | $198 |
| GPS | [u-blox NEO-M9N](https://www.u-blox.com/en/product/neo-m9n-module) | 1 | $30 |
| Leak sensor | [BlueRobotics SOS](https://bluerobotics.com/store/sensors-cameras/leak-sensor/sos-leak-sensor/) | 1 | $35 |
| Battery | 3S3P Samsung 35E + Daly BMS | 1 | $50–65 |

### 9.3 Mechanical & hull

| Part | Model | Qty | Est. cost |
|---|---|---|---|
| Hull tube | [BlueRobotics 4″ WTE 300mm](https://bluerobotics.com/store/watertight-enclosures/locking-series/wte-locking-tube-r1-vp/) | 1 | $40–60 |
| End caps + flanges | [BlueRobotics aluminum](https://bluerobotics.com/store/watertight-enclosures/) | 2 | $80–120 |
| Penetrators | [BlueRobotics WetLink M10](https://bluerobotics.com/store/cables-connectors/penetrators/wlp-vp/) | 6–8 | $80–136 |
| Wings | SLA resin, SD7003, 71 mm chord | 2 | $30–60 |
| Wing clamp + fairing | 3D-printed clamshell | 1 | $10–20 |
| Drop weight | Nichrome burn wire + ballast | 1 | $25 |
| Nose + tail | 3D-printed streamlined | 2 | $15–30 |

**Total: $2,100–$3,300** (excluding Jetson SOM). Add ~$250–300 for Iridium satellite comms.

---

## 10. Open design decisions

| Decision | Options | Impact |
|---|---|---|
| Single vs dual pump | 1× MGD1000S vs 2× | Agility: 3–7 s vs 1.5–3.5 s transitions |
| Valve architecture | 4× 2-way NC vs 4× 3-way NC | 3-way enables fail-safe drain on power loss |
| Jetson variant | Orin Nano (40 TOPS) vs Orin NX (70 TOPS) | NX adds NVENC + DLA + eMMC |
| Jetson carrier | Auvidea JNX120S vs Hadron | Size vs I/O |
| Comms | LoRa only vs LoRa + Iridium | Cost vs open-water safety |
| Hull length | 300 mm vs 400 mm tube | Internal volume vs drag |
| PCB fab | JLCPCB assembly vs hand-solder | Cost vs flexibility |

---

## 11. Known risks

| Risk | Severity | Mitigation |
|---|---|---|
| Oil thermal expansion | Med | NTC + IEKF temperature state + MPC compensation |
| Entrained air in oil | High | Vacuum degas, bottom-up fill, bleed valves |
| Bladder biofouling | Med | Anti-fouling coating, mission <1 week |
| Single bladder leak | High | Reservoir sensing, IEKF residuals, drop-weight abort |
| M7 core failure | Med | M4 safety supervisor resets M7, surfaces if unrecoverable (§5.4) |
| Jetson hang | Med | M4 kills Jetson, M7 continues or surfaces |
| Custom PCB first-spin | Med | Prototype on Nucleo-H755ZI-Q first |

---

## 12. Next steps

- Finalize open design decisions (section 10) — team call
- Order long-lead: TCS pump, Gems valves, BlueRobotics hull kit, Jetson carrier
- **Nick:** Zephyr AMP bringup on Nucleo-H755ZI-Q — M7/M4 heartbeat + HSEM + shared SRAM
- **Nick:** IEKF skeleton on M7, M4 safety supervisor with watchdog + emergency surface
- **Nick:** Jetson pipeline — JetPack 6.x, CSI camera, UART bridge to H755
- **Nick:** KiCad schematic for custom flight controller PCB
- **Pavel:** Hull CAD — internal layout, penetrator placement, wing mount design
- **Dmitry:** Benchtop hydraulic breadboard — pump + valve + bladder, flow vs pressure
- **Daniil:** SD7003 wing CAD (SLA), XFOIL validation at Re 50k, clamp/fairing
- **Milestone 1:** Benchtop buoyancy engine with 4-channel differential control via Nucleo
- **Milestone 2:** Pool test — sealed hull with wings, manual dive/climb/turn, IMU log
- **Milestone 3:** Autonomous waypoint following in pool with Jetson path planner
