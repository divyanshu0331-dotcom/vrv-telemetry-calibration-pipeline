# VRV Telemetry Pipeline: Automated NTC Thermistor Calibration & Physical Anomaly Detection Engine

## 📌 Project Overview
In modern VRV (Variable Refrigerant Volume) HVAC systems, micro-faults in physical sensor infrastructure—such as moisture ingress causing gradual calibration drift—frequently trigger cascading system inefficiencies or premature compressor lockouts. 

This repository contains an end-to-end data validation pipeline that ingests raw component telemetry streams (resistance/voltage), applies programmatic **curve-fitting numerical methods** to derive real-time thermodynamic properties, and uses boundary-envelope verification to isolate physical hardware anomalies before they cause field system failure modes.

## 🛠️ Data Stack & Engineering Concepts
*   **Language:** Python 3.x
*   **Numerical & Statistical Libraries:** `NumPy`, `Pandas`, `SciPy (Optimize)`
*   **Data Visualization:** `Matplotlib`
*   **Domain Engineering Logic:** Steinhart-Hart Thermodynamic Equations, Physical Limit Enveloping, Sensor Drift Isolation.

---

## 📈 System Architecture & Methodology

```
[Raw Component Telemetry] ──> [SciPy Calibration Engine] ──> [Continuous Interpolation] ──> [Boundary Evaluator] ──> [Flagged Alerts]
  (Resistance Streams)          (Fitted Steinhart-Hart)        (Daikin Spec Envelopes)         (Dynamic Multi-Checks)     (Drift / Shorts)
```

### 1. Programmatic Curve Fitting (Calibration Engine)
Instead of relying on hardcoded static approximations, the calibration pipeline leverages a deterministic physical model based on the **Steinhart-Hart Equation**:

$$\frac{1}{T} = A + B \cdot \ln(R) + C \cdot (\ln(R))^3$$

Using factory specification limits derived from baseline component R-T (Resistance vs. Temperature) delivery characteristics, the module runs a non-linear least-squares curve fit (`scipy.optimize.curve_fit`) to compute custom calibration coefficients ($A, B, C$) tailored to the unique manufacturing batch.

### 2. Physical Envelope Interception (Anomaly Detection)
Once transformed from raw electrical resistance ($k\Omega$) to absolute thermal state ($^\circ\text{C}$), telemetry is cross-referenced against factory compliance parameters:
*   **Drift / Component Degradation:** Flags anomalies if component readings stray beyond contract lower or upper boundary envelopes, isolating physical insulation degradation without halting operations.
*   **Critical Fault Intercepts:** Detects catastrophic events, mapping near-zero resistance to short circuits and infinite resistance thresholds to open circuits/disconnections.

---

## 🚀 Execution & Implementation

### Prerequisites
```bash
pip install numpy pandas scipy matplotlib
```

### Key Python Module Snippet
```python
# Extract from the target calibration step
def steinhart_hart_model(R, A, B, C):
    lnR = np.log(R)
    return 1.0 / (A + B * lnR + C * (lnR**3))

# Execute optimization routine against raw baseline specification parameters
popt, _ = curve_fit(f=steinhart_hart_model, xdata=df_spec['mean_ohm'], ydata=df_spec['temp_k'])
A_calibrated, B_calibrated, C_calibrated = popt
```

---

## 📊 Sample Production Output Stream

Upon deployment, the pipeline processes high-frequency continuous data logs and returns structured diagnostic tables:

```text
LIVE VRV SYSTEM TELEMETRY OUTPUT STREAM REPORT
          Timestamp    Raw_kΩ  Calibrated_Temp_°C                                        Status
2026-10-07 09:00:00    65.750                0.00                                        Normal
2026-10-07 09:01:00    39.960               10.00                                        Normal
2026-10-07 09:02:00    20.000               25.00                                        Normal
2026-10-07 09:03:00    18.200               27.30   ANOMALY: Negative Drift / Sub-spec Variance
2026-10-07 09:04:00    10.620               40.00                                        Normal
2026-10-07 09:06:00     3.100               73.80   ANOMALY: Positive Drift / Component Degradation
2026-10-07 09:07:00     0.020              211.23            CRITICAL: Short Circuit Intercepted
2026-10-07 09:09:00  1500.000              -55.80          CRITICAL: Open Circuit / Disconnected
```

## 💡 Key Business Impact
1.  **Reduces Maintenance Overhead:** Shifting field operations from reactive servicing to automated conditional maintenance by flagging sensor degradation long before system lockout faults trigger.
2.  **Optimizes Component Sourcing Lifecycle:** Aggregating out-of-envelope drift profiles helps QA teams trace poor manufacturing quality back to specific component batches or localization suppliers.
3.  **Closes R&D Simulation Loop:** Feeds real-world, calibrated thermodynamic data distributions directly back to hardware design teams to optimize mechanical computational models.