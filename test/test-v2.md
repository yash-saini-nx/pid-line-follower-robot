# ENGINEERING TEST LOG
**Document ID:** LFR-PID-002
**Subject:** PID Parameter Tuning - 5-Channel Analog IR Array
**Target Platform:** Differential Drive Line Follower

## 1. System Configuration
* **Sensor Array:** 5-Channel Analog IR Reflectance Array (e.g., TCRT5000 or similar).
* **ADC Resolution:** 10-bit (0-1023 per channel).
* **Position Algorithm:** Weighted average. Sensor outputs are normalized and multiplied by channel weights (e.g., 0, 1000, 2000, 3000, 4000).
* **Setpoint:** 2000 (Center sensor perfectly aligned over the line).
* **Error Calculation:** `Error = Current Position - Setpoint` (Operational Range: -2000 to +2000).

## 2. Tuning Methodology and Observations
The tuning process utilized empirical step-response testing to observe the chassis dynamics, motor deadband characteristics, and the high-resolution analog error gradient.

### Test Iteration 1: Proportional Only (Under-reactive)
* **Parameters:** `Kp = 0.15`, `Ki = 0.000`, `Kd = 0.00`
* **System Behavior:** The system exhibited a heavily overdamped response. The chassis successfully tracked straight line segments but failed to generate sufficient control effort during high-angle deviations.
* **Error Profile:** Error grew unbounded (>2000) on curve entry, resulting in complete loss of line acquisition.

### Test Iteration 2: Proportional Only (Over-reactive)
* **Parameters:** `Kp = 0.60`, `Ki = 0.000`, `Kd = 0.00`
* **System Behavior:** The system maintained line acquisition but exhibited continuous, undamped oscillations (hunting) across the longitudinal axis. 
* **Error Profile:** Sustained high-frequency alternating error readings (peak-to-peak amplitude of approx. +1500 to -1500).

### Test Iteration 3: Introduction of Derivative Damping
* **Parameters:** `Kp = 0.40`, `Ki = 0.000`, `Kd = 0.50`
* **System Behavior:** Significant reduction in oscillation amplitude. The system stabilized rapidly on straightaways but exhibited marginal overshoot exiting acute angles.
* **Error Profile:** Damped sine wave decay upon step disturbance (+1200, -400, +100, 0).

### Test Iteration 4: PD Tuning Refinement
* **Parameters:** `Kp = 0.32`, `Ki = 0.000`, `Kd = 1.00`
* **System Behavior:** Critically damped response. The chassis tracked dynamic changes cleanly without oscillation. However, telemetry indicated a persistent steady-state error on continuous, long-radius curves.
* **Error Profile:** Maintained a non-zero static offset (approx. +150 to +200) during sustained radius tracking.

### Test Iteration 5: Final PID Implementation
* **Parameters:** `Kp = 0.32`, `Ki = 0.001`, `Kd = 1.00`
* **System Behavior:** Optimal control achieved. The integral term successfully accumulated the steady-state error observed in Iteration 4, driving the control effort higher and forcing the system back to the exact 2000 setpoint during extended curves.
* **Error Profile:** Rapid convergence to 0 error with tight bound maintenance (+/- 15 positional tolerance).

## 3. Step Response Telemetry (Sample Data)

The following table represents simulated telemetry of the analog error value over time immediately following an acute track angle change. Values represent the calculated deviation from the 2000 setpoint.

| Time Step (ms) | Iteration 2 (Kp=0.6, Kd=0) | Iteration 5 (Kp=0.32, Kd=1, Ki=0.001) |
| :--- | :--- | :--- |
| **0** (Disturbance) | 2000 | 2000 |
| **50** | -1650 | 850 |
| **100** | 1580 | 120 |
| **150** | -1420 | -45 |
| **200** | 1390 | 10 |
| **250** | -1350 | 0 |
| **300** | 1300 | 0 |

**Conclusion:** The transition to a weighted average 5-channel analog input provides much higher resolution error tracking compared to a discrete digital array. The final tuned parameters (`Kp=0.32`, `Kd=1.00`, `Ki=0.001`) provide a stable, critically damped response that fully leverages the analog sensor gradient.
