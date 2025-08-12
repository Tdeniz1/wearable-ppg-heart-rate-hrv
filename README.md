# Wearable PPG Heart Rate & HRV (Synthetic Data)

## Overview

This notebook is a compact, end-to-end demo of a wearable heart-rate pipeline using synthetic data. It creates a realistic **photoplethysmogram (PPG)** signal like what a smartwatch optical sensor records, then processes it to produce heart rate and heart rate variability (HRV) metrics with clear plots.

**What the code does**
1. **Generate a synthetic PPG signal**
   - Builds a timeline at 100 Hz for 5 minutes.
   - Creates beat times from a target mean heart rate with small random variability by sampling RR intervals and taking a cumulative sum.
   - For each beat, adds a pulse kernel with a fast rise and slow decay to mimic the systolic upstroke and diastolic falloff.
   - Adds realistic artifacts:
     - Respiratory baseline wander at about 0.25 Hz.
     - Low frequency drift and occasional spikes to simulate motion.
     - White noise to simulate sensor noise.
   - The result is a raw PPG that looks and behaves like real wearable data.

2. **Preprocess the signal**
   - Detrends to remove slow drift.
   - Applies a 0.5 to 5 Hz Butterworth bandpass filter to isolate the cardiac component of the waveform and suppress baseline and high-frequency noise.
   - Shows a side-by-side plot of raw versus filtered so you can see the improvement.

3. **Detect beats and compute heart rate**
   - Uses `scipy.signal.find_peaks` with a minimum distance and prominence threshold so it does not double count or chase noise.
   - Converts the time between consecutive detected peaks to instantaneous heart rate in beats per minute.
   - Plots heart rate across the whole session to verify it looks stable and plausible.

4. **Compute HRV metrics**
   - **Time domain**: 
     - SDNN is the standard deviation of RR intervals in seconds. 
     - RMSSD is the root mean square of successive differences of RR intervals. 
     - Both are commonly reported in research and consumer products.
   - **Frequency domain**:
     - Interpolates RR intervals to an even grid at 4 Hz and runs Welch power spectral density.
     - Integrates power in standard bands:
       - LF 0.04 to 0.15 Hz
       - HF 0.15 to 0.40 Hz
     - Reports LF, HF, and LF to HF ratio.
   - **Nonlinear view**:
     - Plots a Poincaré scatter of RR_n against RR_{n+1} to visualize rhythm stability.

5. **Export results and artifacts**
   - Saves `ppg_timeseries.csv` with time, raw PPG, and filtered PPG columns.
   - Saves `ppg_metrics.json` with mean heart rate, SDNN, and RMSSD for quick inspection and for inclusion in the repo.

**Why synthetic**
- No privacy risk and no dataset hunting. 
- Reproducible on any machine, including Colab, with standard scientific Python libraries.
- Still realistic enough to demonstrate signal processing, detection, and HRV workflows used in wearables and research.

**What the plots show**
- Raw vs Filtered PPG: effect of preprocessing
- Detected Beats: correctness of the peak detector
- Heart Rate over Time: the metric users and clinicians care about
- HRV Power Spectral Density: autonomic balance features in LF and HF bands
- Poincaré Plot: quick visual of variability and rhythm stability

**Limitations to be aware of**
- The data are simulated and will not include all edge cases like severe motion, poor perfusion, or arrhythmias.
- Peak detection thresholds are tuned for this synthetic signal and may need adjustment for real sensors.
- Frequency domain HRV is sensitive to interpolation choices and windowing. The implementation follows common defaults but is simplified for clarity.

**Tech stack**
- Python, NumPy, SciPy, Matplotlib, Pandas. Runs out of the box in Google Colab.

---

## Features
- **Signal Simulation**: Generates a 5-minute PPG signal with physiological artifacts (respiration drift, motion noise, white noise).
- **Preprocessing**: Detrending + 0.5–5 Hz Butterworth bandpass filter to isolate pulse frequency band.
- **Beat Detection**: Peak detection with prominence/distance criteria to find heartbeats.
- **Heart Rate Calculation**: Instantaneous HR over time derived from RR intervals.
- **HRV Metrics**:
  - **Time-domain**: SDNN, RMSSD.
  - **Frequency-domain**: LF/HF power ratio via Welch PSD.
  - **Poincaré plot** for RR interval variability.
- **Visualizations**:
  1. Raw vs. Filtered PPG (10 s).
  2. Detected Beats (10 s window).
  3. Heart Rate over Time.
  4. HRV Power Spectral Density.
  5. Poincaré Plot.
- **Outputs**:  
  - `ppg_timeseries.csv` — raw & filtered signals.
  - `ppg_metrics.json` — mean HR, SDNN, RMSSD.

---

## Plots

### 1. Raw vs. Filtered PPG (10 s)
This plot compares the **unprocessed (raw)** simulated PPG signal to the **bandpass-filtered** version over a 10-second window.  
- The raw signal includes baseline wander from respiration, low-frequency drift from motion, occasional spikes, and white noise — all common in real wearable sensor data.  
- The filtered signal has these artifacts largely removed, isolating the pulsatile waveform corresponding to heartbeats.  
- This step demonstrates the importance of preprocessing in biomedical signal analysis — without it, heartbeat detection would be highly unreliable.

### 2. Detected Beats (10 s)
Displays the filtered PPG from the same time window as above, but with **markers** placed on each detected systolic peak.  
- These peaks represent the moments when blood volume in the tissue is at its maximum during each cardiac cycle.  
- The peak detection algorithm uses both **minimum distance** (to avoid double-counting beats) and **prominence thresholds** (to avoid detecting noise).  
- This plot visually validates that the detection logic is working and is an essential check before calculating HR or HRV.

### 3. Heart Rate over Time
Plots the **instantaneous heart rate** (in beats per minute) calculated from the inverse of RR intervals (time between consecutive detected peaks).  
- This is essentially what a wearable device’s heart rate display would show over the course of the 5-minute simulation.  
- A stable HR line indicates consistent beat detection, while sharp drops or spikes may reveal signal disturbances or physiological variability.  
- This plot connects the raw signal processing to a metric that is both clinically and commercially relevant.

### 4. HRV Power Spectral Density
A **frequency-domain** representation of heart rate variability, calculated using **Welch’s method** on the evenly resampled RR interval series.  
- The power spectrum is divided into:
  - **Low Frequency (LF)**: 0.04–0.15 Hz, associated with both sympathetic and parasympathetic modulation.
  - **High Frequency (HF)**: 0.15–0.40 Hz, associated with parasympathetic (vagal) activity and respiratory sinus arrhythmia.  
- The **LF/HF ratio** is often interpreted as a measure of autonomic balance.  
- This plot is critical in HRV research and is widely used in sports science, stress monitoring, and medical diagnostics.

### 5. Poincaré Plot
A scatter plot of each RR interval (**RRₙ**) against the next consecutive RR interval (**RRₙ₊₁**).  
- Produces an ellipse-like shape when the rhythm is steady, with wider dispersion indicating greater variability.  
- **Tight, narrow clouds** suggest low HRV (often seen in stress or certain disease states), while **broader, more dispersed clouds** indicate high HRV (often seen in well-conditioned or relaxed states).  
- This plot provides a quick, intuitive way to assess rhythm stability and variability without delving into statistical calculations.

---

## Requirements
- Python 3.8+
- numpy
- scipy
- pandas
- matplotlib

All are pre-installed in Google Colab.

---

## Use Cases
- Demonstrating biomedical signal processing skills.
- Showcasing end-to-end wearable device data analysis.
- Teaching HR/HRV analysis concepts.

---

## License
MIT License — see [LICENSE](LICENSE) for details.
