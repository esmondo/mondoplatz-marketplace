# Signal Processing for Neural Data

## Overview

Signal processing transforms raw neural recordings into meaningful information. This guide covers preprocessing, time-domain analysis, frequency-domain analysis, and time-frequency methods for EEG, MEG, fMRI, EMG, and other biosignals.

## Common Biosignal Characteristics

### EEG (Electroencephalography)
- **Frequency range**: 0.5-100 Hz (typical focus: 0.5-50 Hz)
- **Amplitude**: 10-100 μV
- **Sampling rate**: 250-1000 Hz (typical: 250-512 Hz)
- **Channels**: Consumer: 1-8, Research: 32-256+
- **Spatial resolution**: Low (~5-10 cm)
- **Temporal resolution**: Excellent (~1 ms)

**Common artifacts**: Eye blinks (~100-200 μV), muscle activity (>30 Hz), heart (ECG), movement

### MEG (Magnetoencephalography)
- **Frequency range**: 0.1-100+ Hz
- **Amplitude**: 10-1000 fT (femtotesla)
- **Sampling rate**: 600-6000 Hz
- **Channels**: 100-300+
- **Spatial resolution**: Better than EEG (~2-3 cm)
- **Temporal resolution**: Excellent (~1 ms)

**Advantages over EEG**: Less affected by skull/scalp, better source localization

### fMRI (Functional MRI)
- **Measures**: Blood oxygen level dependent (BOLD) signal
- **Temporal resolution**: Poor (~2 seconds for TR)
- **Spatial resolution**: Excellent (1-3 mm³)
- **Signal**: Indirect measure of neural activity (hemodynamic response)
- **Delay**: 4-6 second lag from neural activity

**Analysis approaches**: GLM (general linear model), connectivity analysis, pattern classification

### EMG (Electromyography)
- **Frequency range**: 10-500 Hz (main power: 50-150 Hz)
- **Amplitude**: 0.1-5 mV (surface), higher for needle
- **Sampling rate**: 1000-4000 Hz
- **Applications**: Muscle activity detection, artifact removal from EEG, motor control

### ECG (Electrocardiogram)
- **Frequency range**: 0.5-100 Hz (QRS complex: 10-40 Hz)
- **Amplitude**: 0.5-4 mV
- **Sampling rate**: 250-1000 Hz
- **Heart rate**: Typically 60-100 bpm at rest
- **HRV analysis**: Time-domain and frequency-domain measures of autonomic function

### GSR/EDA (Galvanic Skin Response / Electrodermal Activity)
- **Components**: Tonic (baseline) and phasic (responses)
- **Sampling rate**: 10-100 Hz (slow signal)
- **Measures**: Sympathetic nervous system activity, arousal
- **Applications**: Stress, emotion, cognitive load assessment

## Preprocessing Pipeline

### 1. Data Import and Inspection

**Load raw data**:
```python
# MNE-Python example
import mne
import numpy as np

# Load raw EEG data
raw = mne.io.read_raw_edf('data.edf', preload=True)

# Inspect basic properties
print(f"Sampling rate: {raw.info['sfreq']} Hz")
print(f"Channels: {raw.ch_names}")
print(f"Duration: {raw.times[-1]:.2f} seconds")

# Visualize raw data
raw.plot(duration=10, n_channels=30, scalings='auto')
```

**MATLAB equivalent**:
```matlab
% Using EEGLAB
EEG = pop_loadset('data.set');
eeglab redraw  % Visualize in EEGLAB GUI
```

**Initial inspection checklist**:
- Verify sampling rate is appropriate
- Check channel names and locations
- Identify obvious artifacts or bad channels
- Confirm recording duration and events

### 2. Channel Selection and Re-referencing

**Bad channel detection**:
- Visual inspection for flat or noisy channels
- Statistical outliers (amplitude, variance)
- Correlation with neighbors

```python
# Mark bad channels
raw.info['bads'] = ['Fp1', 'T8']  # Example bad channels

# Interpolate bad channels (optional)
raw.interpolate_bads(reset_bads=True)
```

**Re-referencing strategies**:
- **Average reference**: Subtract mean of all electrodes (most common)
- **Mastoid reference**: Reference to M1, M2, or average of both
- **Linked ears**: Average of left and right ear electrodes
- **Laplacian**: Spatial derivative (each electrode minus weighted neighbors)

```python
# Average reference
raw.set_eeg_reference('average', projection=True)

# Mastoid reference
raw.set_eeg_reference(['M1', 'M2'])
```

### 3. Filtering

**High-pass filtering** (remove slow drifts):
- **Purpose**: Remove DC offset and slow drifts
- **Typical cutoff**: 0.1-1 Hz
- **Trade-off**: Higher cutoff removes more artifact but may distort slow oscillations

```python
# High-pass filter at 0.5 Hz
raw.filter(l_freq=0.5, h_freq=None, fir_design='firwin')
```

**Low-pass filtering** (remove high-frequency noise):
- **Purpose**: Remove muscle artifact, electrical noise
- **Typical cutoff**: 40-50 Hz for EEG (lower for slow signals)
- **Anti-aliasing**: Already applied during recording

```python
# Low-pass filter at 40 Hz
raw.filter(l_freq=None, h_freq=40, fir_design='firwin')
```

**Notch filtering** (remove line noise):
- **Purpose**: Remove 50 Hz or 60 Hz electrical interference
- **Application**: Only if line noise present and not removed by other methods
- **Caution**: Can introduce artifacts, use sparingly

```python
# Notch filter at 60 Hz (US) or 50 Hz (Europe)
raw.notch_filter(freqs=60)
```

**Bandpass filtering** (select frequency range of interest):
```python
# Bandpass filter 1-40 Hz
raw.filter(l_freq=1.0, h_freq=40.0, fir_design='firwin')
```

**Filter design considerations**:
- **FIR vs IIR**: FIR filters (linear phase) preferred for neuroscience
- **Filter order**: Higher order = sharper cutoff but more computation
- **Zero-phase filtering**: Apply filter forwards and backwards to avoid phase distortion
- **Edge effects**: Filter artifacts at beginning/end of recording

### 4. Artifact Removal

**Independent Component Analysis (ICA)**:
- Separates signals into independent sources
- Identify artifact components (eye blinks, heartbeat, muscle)
- Remove artifact components and reconstruct clean signal

```python
from mne.preprocessing import ICA

# Fit ICA (typically 20-30 components)
ica = ICA(n_components=25, random_state=97, max_iter=800)
ica.fit(raw, picks='eeg')

# Find eye blink components automatically
eog_indices, eog_scores = ica.find_bads_eog(raw, ch_name='Fp1')
ica.exclude = eog_indices

# Apply ICA to remove artifacts
raw_clean = ica.apply(raw.copy())
```

**Automated artifact rejection**:
- Reject epochs/trials with amplitude > threshold
- Reject based on gradient, variance, or other metrics

```python
# Example: Reject epochs with amplitude > 100 μV
reject_criteria = dict(eeg=100e-6)  # 100 μV in volts
epochs_clean = mne.Epochs(raw, events, reject=reject_criteria)
```

**Manual artifact rejection**:
- Visual inspection of data
- Mark bad segments for exclusion
- Time-consuming but most accurate

### 5. Epoching / Segmentation

**Event-related analysis**:
- Extract time windows around specific events
- Baseline correction (subtract pre-stimulus interval)

```python
# Define events (e.g., stimulus presentations)
events = mne.find_events(raw, stim_channel='STI 014')

# Create epochs: -0.2 to 1.0 seconds around events
epochs = mne.Epochs(raw, events, event_id={'stimulus': 1}, 
                    tmin=-0.2, tmax=1.0, baseline=(-0.2, 0))
```

**Resting-state analysis**:
- Segment continuous data into fixed windows
- Typical window: 2-4 seconds with 50% overlap

```python
# Create fixed-length epochs for resting state
epochs = mne.make_fixed_length_epochs(raw, duration=2.0, overlap=1.0)
```

## Time-Domain Analysis

### Event-Related Potentials (ERPs)

**Concept**: Average response to repeated stimuli reveals time-locked brain activity.

**Procedure**:
1. Epoch data around events
2. Average across trials
3. Identify ERP components (P1, N1, P2, N2, P300, etc.)

```python
# Compute ERP
evoked = epochs.average()

# Plot ERP
evoked.plot(spatial_colors=True, gfp=True)

# Extract peak amplitude and latency
peak_time = evoked.get_peak(tmin=0.3, tmax=0.6, mode='pos')[1]
peak_amplitude = evoked.get_peak(tmin=0.3, tmax=0.6, mode='pos')[0]
```

**Common ERP components**:
- **P1** (~100 ms): Early sensory processing
- **N1** (~150 ms): Attention, sensory gating
- **P2** (~200 ms): Stimulus evaluation
- **N2** (~200-300 ms): Conflict monitoring, inhibition
- **P300** (~300-600 ms): Target detection, memory updating
- **N400** (~400 ms): Semantic processing
- **P600** (~600 ms): Syntactic processing

### Amplitude and Waveform Analysis

**Peak detection**:
```python
# Find local maxima/minima
from scipy.signal import find_peaks

signal = epochs.get_data()[0, 0, :]  # First trial, first channel
peaks, _ = find_peaks(signal, height=0)
```

**Root mean square (RMS)**:
```python
# Compute RMS power over time window
rms = np.sqrt(np.mean(signal**2))
```

**Area under curve**:
```python
from scipy.integrate import trapz

# Integrate signal over time
area = trapz(signal, dx=1/epochs.info['sfreq'])
```

## Frequency-Domain Analysis

### Fourier Transform

**Concept**: Decompose time-series into constituent frequencies.

**Fast Fourier Transform (FFT)**:
```python
from scipy.fft import fft, fftfreq

# Compute FFT
signal = epochs.get_data()[0, 0, :]
fft_vals = fft(signal)
fft_freq = fftfreq(len(signal), 1/epochs.info['sfreq'])

# Power spectrum (one-sided)
power = np.abs(fft_vals)**2
freq = fft_freq[:len(fft_freq)//2]
power = power[:len(power)//2]

# Plot power spectrum
import matplotlib.pyplot as plt
plt.plot(freq, 10*np.log10(power))  # Convert to dB
plt.xlabel('Frequency (Hz)')
plt.ylabel('Power (dB)')
```

**Welch's method** (more robust):
```python
from scipy.signal import welch

# Compute PSD using Welch's method
frequencies, psd = welch(signal, fs=epochs.info['sfreq'], 
                         nperseg=256, noverlap=128)

plt.semilogy(frequencies, psd)
plt.xlabel('Frequency (Hz)')
plt.ylabel('Power Spectral Density (μV²/Hz)')
```

### Power Spectral Density (PSD)

**Band power calculation**:
```python
# Extract power in specific frequency bands
def band_power(psd, frequencies, band):
    """Calculate power in frequency band."""
    idx = np.logical_and(frequencies >= band[0], frequencies <= band[1])
    return np.trapz(psd[idx], frequencies[idx])

# Define frequency bands
bands = {
    'delta': (0.5, 4),
    'theta': (4, 8),
    'alpha': (8, 13),
    'beta': (13, 30),
    'gamma': (30, 50)
}

# Calculate band powers
band_powers = {band: band_power(psd, frequencies, freq_range) 
               for band, freq_range in bands.items()}

print(band_powers)
```

**Relative band power**:
```python
# Express each band as proportion of total power
total_power = sum(band_powers.values())
relative_powers = {band: power/total_power 
                   for band, power in band_powers.items()}
```

### Connectivity Analysis

**Coherence** (frequency-domain correlation):
```python
from mne.connectivity import spectral_connectivity_epochs

# Compute coherence between all channel pairs
connectivity = spectral_connectivity_epochs(
    epochs, method='coh', mode='multitaper',
    sfreq=epochs.info['sfreq'], fmin=8, fmax=13, 
    faverage=True, n_jobs=1)
```

**Phase-locking value (PLV)**:
```python
# Measure phase synchronization
connectivity_plv = spectral_connectivity_epochs(
    epochs, method='plv', mode='multitaper',
    sfreq=epochs.info['sfreq'], fmin=8, fmax=13)
```

## Time-Frequency Analysis

### Short-Time Fourier Transform (STFT)

**Concept**: Apply FFT to overlapping time windows to see how frequency content changes over time.

```python
from scipy.signal import spectrogram

# Compute spectrogram
frequencies, times, Sxx = spectrogram(signal, 
                                      fs=epochs.info['sfreq'],
                                      nperseg=256, noverlap=200)

# Plot spectrogram
plt.pcolormesh(times, frequencies, 10*np.log10(Sxx), shading='gouraud')
plt.ylabel('Frequency (Hz)')
plt.xlabel('Time (s)')
plt.colorbar(label='Power (dB)')
```

**Time-frequency resolution trade-off**:
- Longer windows: Better frequency resolution, worse time resolution
- Shorter windows: Better time resolution, worse frequency resolution

### Wavelet Transform

**Concept**: Use wavelets (oscillating waveforms) instead of sine waves for better time-frequency localization.

```python
from mne.time_frequency import tfr_morlet

# Compute time-frequency representation using Morlet wavelets
freqs = np.logspace(*np.log10([4, 40]), num=20)  # 4-40 Hz
n_cycles = freqs / 2  # Number of cycles per wavelet

power = tfr_morlet(epochs, freqs=freqs, n_cycles=n_cycles, 
                   use_fft=True, return_itc=False, average=True)

# Plot
power.plot([0], baseline=(-0.5, 0), mode='logratio', 
           title='Time-Frequency Power')
```

**Wavelet types**:
- **Morlet**: Gaussian-windowed sine wave (most common in neuroscience)
- **Mexican hat**: Second derivative of Gaussian
- **Daubechies**: Discrete wavelets for multi-resolution analysis

### Event-Related Spectral Perturbation (ERSP)

**Concept**: Changes in spectral power time-locked to events, averaged across trials.

```python
# ERSP is computed automatically with baseline correction
power.plot([0], baseline=(-0.5, 0), mode='percent', 
           title='ERSP (% change from baseline)')
```

**Interpretation**:
- **Event-related synchronization (ERS)**: Increased power
- **Event-related desynchronization (ERD)**: Decreased power
- Common in motor cortex during movement preparation/execution

### Inter-Trial Coherence (ITC)

**Concept**: Phase consistency across trials, independent of power.

```python
# Compute both power and ITC
power, itc = tfr_morlet(epochs, freqs=freqs, n_cycles=n_cycles, 
                        use_fft=True, return_itc=True, average=True)

# Plot ITC
itc.plot([0], title='Inter-Trial Coherence')
```

## Practical Considerations

### Sampling Rate Selection

**Nyquist theorem**: Must sample at ≥2× highest frequency of interest.
- For 50 Hz max: Need ≥100 Hz sampling rate
- Practical: Use 4-5× highest frequency for safety (200-250 Hz for 50 Hz signal)
- Common EEG rates: 250, 500, 1000 Hz

### Window Length and Overlap

**FFT window length**:
- Longer window: Better frequency resolution
- Shorter window: Better captures non-stationarities
- Typical: 1-4 seconds (256-1024 samples at 256 Hz)
- Frequency resolution = sampling_rate / window_length

**Overlap**:
- Typical: 50% overlap
- More overlap: Smoother results, more computation
- Less overlap: Faster computation, may miss transient events

### Baseline Correction

**Purpose**: Remove pre-stimulus activity to isolate event-related changes.

**Methods**:
- **Subtraction**: Signal - baseline_mean
- **Division**: Signal / baseline_mean  
- **Percent change**: (Signal - baseline_mean) / baseline_mean × 100
- **Decibel**: 10 * log10(Signal / baseline_mean)

```python
# Apply baseline correction to epochs
epochs.apply_baseline((-0.2, 0))  # Use -200 to 0 ms as baseline
```

### Multiple Comparisons Correction

**Problem**: Testing many channels/frequencies/time points increases false positives.

**Solutions**:
- **Bonferroni**: Divide α by number of comparisons (very conservative)
- **FDR** (False Discovery Rate): Less conservative, controls proportion of false positives
- **Cluster-based permutation**: Find clusters of effects, test cluster-level significance
- **Non-parametric**: Permutation tests avoid assumptions about distribution

## Common Pitfalls and Solutions

### 1. Filter artifacts
- **Problem**: Ringing, edge effects
- **Solution**: Pad data before filtering, use appropriate filter design

### 2. Insufficient preprocessing
- **Problem**: Artifacts contaminate results
- **Solution**: Always inspect data, use ICA, reject bad trials

### 3. Inappropriate baseline
- **Problem**: Baseline contains task-related activity
- **Solution**: Use longer pre-stimulus interval, verify baseline is stable

### 4. Circular analysis
- **Problem**: Using same data for selection and testing
- **Solution**: Independent data for validation, cross-validation

### 5. Ignoring individual differences
- **Problem**: Group averages obscure individual patterns
- **Solution**: Report individual results, consider personalized approaches

## Hardware-Specific Considerations

### Consumer EEG (e.g., Muse)
- Limited channels (2-8)
- Lower sampling rates (256 Hz typical)
- More prone to artifacts
- Focus on robust, established metrics
- See `tools-and-packages.md` for specific hardware APIs

### Research-Grade EEG
- Many channels (32-256+)
- Higher sampling rates (500-1000+ Hz)
- Source localization possible
- More preprocessing options
- Requires more expertise

### Real-Time Processing
- **Latency constraints**: Process data as it arrives
- **Computational efficiency**: Use optimized algorithms
- **Adaptive filtering**: Update filters online
- **Artifact detection**: Real-time artifact removal critical
- See `bci-systems.md` for real-time pipelines

## Further Reading

- See `tools-and-packages.md` for specific implementation examples
- See `bci-systems.md` for real-time processing pipelines
- See `neuroscience-fundamentals.md` for interpretation of frequency bands
- See `learning-resources.md` for signal processing courses and textbooks
