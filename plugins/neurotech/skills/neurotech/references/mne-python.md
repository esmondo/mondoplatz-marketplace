# MNE-Python Reference

MNE-Python is the de facto standard for EEG/MEG analysis in Python. This reference covers common workflows, functions, and patterns.

## Installation

```bash
pip install mne
# Optional for full functionality
pip install mne[full]
# For 3D visualization
pip install mayavi pyvista pyvistaqt
```

## Core Data Structures

### Raw (Continuous Data)

```python
import mne

# Reading various formats
raw = mne.io.read_raw_edf('file.edf', preload=True)
raw = mne.io.read_raw_bdf('file.bdf', preload=True)
raw = mne.io.read_raw_fif('file.fif', preload=True)
raw = mne.io.read_raw_brainvision('file.vhdr', preload=True)
raw = mne.io.read_raw_eeglab('file.set', preload=True)

# Key attributes
raw.info              # Measurement info (channels, sfreq, etc.)
raw.ch_names          # Channel names
raw.info['sfreq']     # Sampling frequency
raw.times             # Time vector
raw.n_times           # Number of time points

# Get data as numpy array
data = raw.get_data()  # Shape: (n_channels, n_times)
data, times = raw.get_data(return_times=True)

# Selecting channels
raw.pick_channels(['Fz', 'Cz', 'Pz'])
raw.pick_types(eeg=True, exclude='bads')
raw.drop_channels(['EOG1', 'EOG2'])
```

### Epochs (Segmented Trials)

```python
# Find events from trigger channel
events = mne.find_events(raw, stim_channel='STI 014')
# Events array: [[sample, 0, event_id], ...]

# Create event dictionary
event_id = {'target': 1, 'standard': 2}

# Create epochs
epochs = mne.Epochs(
    raw, 
    events, 
    event_id=event_id,
    tmin=-0.2,           # Start time before event
    tmax=0.8,            # End time after event
    baseline=(None, 0),  # Baseline correction window
    preload=True,
    reject=dict(eeg=100e-6)  # Reject epochs > 100 µV
)

# Accessing data
epochs['target']           # Select by condition
epochs[0]                  # First epoch
epochs.get_data()          # Shape: (n_epochs, n_channels, n_times)
epochs.drop_bad()          # Remove rejected epochs
epochs.metadata            # Pandas DataFrame for trial info
```

### Evoked (Averaged ERPs)

```python
# Average epochs
evoked = epochs.average()
evoked_target = epochs['target'].average()

# Key methods
evoked.plot()
evoked.plot_topomap(times=[0.1, 0.2, 0.3])
evoked.plot_joint()

# Get peak
ch, latency, amplitude = evoked.get_peak(
    ch_type='eeg', 
    tmin=0.1, 
    tmax=0.3
)

# Combine evokeds
evoked_diff = mne.combine_evoked(
    [evoked_target, evoked_standard], 
    weights=[1, -1]
)
```

## Preprocessing

### Filtering

```python
# Bandpass filter
raw.filter(l_freq=1, h_freq=40, method='fir')

# Highpass only
raw.filter(l_freq=1, h_freq=None)

# Lowpass only
raw.filter(l_freq=None, h_freq=40)

# Notch filter (power line noise)
raw.notch_filter(freqs=[50, 100, 150])  # EU
raw.notch_filter(freqs=[60, 120, 180])  # US

# Filter parameters
raw.filter(
    l_freq=1, 
    h_freq=40,
    filter_length='auto',
    l_trans_bandwidth='auto',
    h_trans_bandwidth='auto',
    method='fir',
    phase='zero-double',
    fir_window='hamming'
)
```

### Re-referencing

```python
# Average reference
raw.set_eeg_reference('average', projection=True)
raw.apply_proj()

# Specific reference
raw.set_eeg_reference(['M1', 'M2'])  # Linked mastoids
raw.set_eeg_reference(['Cz'])        # Single electrode

# REST reference (requires forward model)
raw.set_eeg_reference('REST', forward=fwd)
```

### Channel Operations

```python
# Set montage (electrode positions)
montage = mne.channels.make_standard_montage('standard_1020')
raw.set_montage(montage)

# Mark bad channels
raw.info['bads'] = ['Fp1', 'Fp2']

# Interpolate bad channels
raw.interpolate_bads(reset_bads=True)

# Rename channels
raw.rename_channels({'EEG001': 'Fz', 'EEG002': 'Cz'})

# Set channel types
raw.set_channel_types({'EOG1': 'eog', 'ECG': 'ecg'})
```

### Artifact Removal with ICA

```python
from mne.preprocessing import ICA

# Fit ICA
ica = ICA(
    n_components=20,      # Number of components
    method='fastica',     # or 'infomax', 'picard'
    random_state=42
)
ica.fit(raw, picks='eeg')

# Visualize components
ica.plot_components()
ica.plot_sources(raw)

# Find EOG artifacts automatically
eog_indices, eog_scores = ica.find_bads_eog(raw)
ica.exclude = eog_indices

# Find ECG artifacts
ecg_indices, ecg_scores = ica.find_bads_ecg(raw)
ica.exclude.extend(ecg_indices)

# Apply ICA (remove artifacts)
raw_clean = ica.apply(raw.copy())

# For epochs
epochs_clean = ica.apply(epochs.copy())
```

### Autoreject (Automatic Epoch Rejection)

```python
from autoreject import AutoReject, get_rejection_threshold

# Get rejection threshold
reject = get_rejection_threshold(epochs)

# Or use AutoReject for per-channel thresholds
ar = AutoReject(n_interpolate=[1, 2, 4], random_state=42)
epochs_clean, reject_log = ar.fit_transform(epochs, return_log=True)

# Visualize rejection
reject_log.plot()
```

## Frequency Analysis

### Power Spectral Density

```python
# Welch's method on Raw
spectrum = raw.compute_psd(method='welch', fmin=1, fmax=40)
spectrum.plot()

# On Epochs
spectrum = epochs.compute_psd(method='welch', fmin=1, fmax=40)
spectrum.plot()

# Get data
psds, freqs = spectrum.get_data(return_freqs=True)
# Shape: (n_epochs, n_channels, n_freqs) or (n_channels, n_freqs)

# Band power extraction
def bandpower(psd, freqs, band):
    low, high = band
    idx = (freqs >= low) & (freqs <= high)
    return psd[..., idx].mean(axis=-1)

theta_power = bandpower(psds, freqs, (4, 8))
alpha_power = bandpower(psds, freqs, (8, 13))
beta_power = bandpower(psds, freqs, (13, 30))
```

### Time-Frequency Analysis

```python
from mne.time_frequency import tfr_morlet, tfr_multitaper

# Morlet wavelets
freqs = np.arange(4, 40, 1)  # Frequencies of interest
n_cycles = freqs / 2.        # Variable cycles

power = tfr_morlet(
    epochs, 
    freqs=freqs, 
    n_cycles=n_cycles,
    return_itc=False,
    average=True
)
power.plot([0])  # Plot channel 0

# With inter-trial coherence
power, itc = tfr_morlet(
    epochs, 
    freqs=freqs, 
    n_cycles=n_cycles,
    return_itc=True
)

# Multitaper method
power = tfr_multitaper(
    epochs, 
    freqs=freqs, 
    n_cycles=n_cycles,
    time_bandwidth=4.0
)

# Event-related spectral perturbation (baseline correction)
power.apply_baseline(baseline=(-0.5, 0), mode='logratio')
```

## Event-Related Potentials

### ERP Analysis

```python
# Grand average across subjects
evokeds = [subj_epochs.average() for subj_epochs in all_epochs]
grand_avg = mne.grand_average(evokeds)

# Plot ERPs
evoked.plot(spatial_colors=True)
evoked.plot_image()

# Compare conditions
mne.viz.plot_compare_evokeds(
    {'target': evoked_target, 'standard': evoked_standard},
    picks='Pz'
)

# Topographic maps at specific times
evoked.plot_topomap(times=[0.1, 0.2, 0.3, 0.4])

# Joint plot (waveforms + topos)
evoked.plot_joint()
```

### Peak Detection

```python
# Find peak in time window
ch_name, latency, amplitude = evoked.get_peak(
    ch_type='eeg',
    tmin=0.25,
    tmax=0.35,
    mode='pos'  # or 'neg', 'abs'
)

# Extract amplitude at specific time
idx = evoked.time_as_index(0.3)
amplitude_at_300ms = evoked.data[:, idx]
```

## Decoding / Machine Learning

### Basic Classification

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.model_selection import cross_val_score
from mne.decoding import Vectorizer

# Prepare data
X = epochs.get_data()  # (n_epochs, n_channels, n_times)
y = epochs.events[:, 2]  # Labels

# Simple pipeline
clf = make_pipeline(
    Vectorizer(),           # Flatten channels x times
    StandardScaler(),
    LinearDiscriminantAnalysis()
)

# Cross-validation
scores = cross_val_score(clf, X, y, cv=5)
print(f"Accuracy: {scores.mean():.2f} (+/- {scores.std():.2f})")
```

### Temporal Decoding (Sliding Window)

```python
from mne.decoding import SlidingEstimator, cross_val_multiscore

# Create time-resolved classifier
clf = make_pipeline(StandardScaler(), LinearDiscriminantAnalysis())
time_decod = SlidingEstimator(clf, n_jobs=-1, scoring='roc_auc')

# Fit and score
scores = cross_val_multiscore(time_decod, X, y, cv=5)
scores = scores.mean(axis=0)  # Average across folds

# Plot
plt.plot(epochs.times, scores)
plt.axhline(0.5, color='k', linestyle='--')
plt.xlabel('Time (s)')
plt.ylabel('AUC')
```

### Common Spatial Patterns (CSP)

```python
from mne.decoding import CSP

# CSP for motor imagery
csp = CSP(n_components=4, reg=None, log=True)

# Pipeline with CSP
clf = make_pipeline(csp, LinearDiscriminantAnalysis())
scores = cross_val_score(clf, X, y, cv=5)

# Visualize CSP patterns
csp.fit(X, y)
csp.plot_patterns(epochs.info)
```

## Source Localization

### Forward Model

```python
# Download fsaverage
fs_dir = mne.datasets.fetch_fsaverage()
subjects_dir = os.path.dirname(fs_dir)

# Source space
src = mne.setup_source_space(
    'fsaverage', 
    spacing='oct6',
    subjects_dir=subjects_dir
)

# BEM model
model = mne.make_bem_model(
    'fsaverage', 
    subjects_dir=subjects_dir
)
bem = mne.make_bem_solution(model)

# Forward solution
fwd = mne.make_forward_solution(
    raw.info, 
    trans='fsaverage',
    src=src, 
    bem=bem
)
```

### Inverse Solution

```python
# Noise covariance from baseline
noise_cov = mne.compute_covariance(
    epochs, 
    tmax=0,
    method='empirical'
)

# Minimum norm estimate
inverse_operator = mne.minimum_norm.make_inverse_operator(
    evoked.info, 
    fwd, 
    noise_cov
)

# Apply inverse
stc = mne.minimum_norm.apply_inverse(
    evoked, 
    inverse_operator,
    lambda2=1/9,
    method='dSPM'
)

# Visualize
brain = stc.plot(
    subjects_dir=subjects_dir,
    hemi='both',
    time_viewer=True
)
```

## Common Workflows

### ERP Study Pipeline

```python
import mne
import numpy as np

# 1. Load data
raw = mne.io.read_raw_edf('data.edf', preload=True)

# 2. Set montage
montage = mne.channels.make_standard_montage('standard_1020')
raw.set_montage(montage)

# 3. Filter
raw.filter(0.1, 30)
raw.notch_filter(50)

# 4. Re-reference
raw.set_eeg_reference('average')

# 5. ICA artifact removal
ica = ICA(n_components=15)
ica.fit(raw)
ica.exclude = ica.find_bads_eog(raw)[0]
raw = ica.apply(raw)

# 6. Create epochs
events = mne.find_events(raw)
epochs = mne.Epochs(raw, events, tmin=-0.2, tmax=0.8,
                    baseline=(-0.2, 0), preload=True)
epochs.drop_bad(reject=dict(eeg=100e-6))

# 7. Average and analyze
evoked = epochs.average()
evoked.plot_joint()
```

### BCI Motor Imagery Pipeline

```python
from mne.decoding import CSP
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.model_selection import cross_val_score
from sklearn.pipeline import Pipeline

# 1. Load and filter for mu/beta
raw.filter(8, 30)

# 2. Create epochs around motor imagery cues
epochs = mne.Epochs(raw, events, event_id={'left': 1, 'right': 2},
                    tmin=0, tmax=4, baseline=None)

# 3. Get data
X = epochs.get_data()
y = epochs.events[:, 2]

# 4. CSP + LDA pipeline
clf = Pipeline([
    ('csp', CSP(n_components=4)),
    ('lda', LinearDiscriminantAnalysis())
])

# 5. Evaluate
scores = cross_val_score(clf, X, y, cv=5)
print(f"Classification accuracy: {scores.mean():.1%}")
```

## Useful Tips

### Memory Management

```python
# Don't preload large files
raw = mne.io.read_raw_edf('large.edf', preload=False)

# Load specific time range
raw.crop(tmin=0, tmax=60).load_data()

# Downsample if needed
raw.resample(250)
```

### Parallel Processing

```python
# Set number of jobs globally
mne.set_config('MNE_USE_CUDA', 'true')  # If GPU available

# Or per function
epochs = mne.Epochs(..., n_jobs=-1)
ica.fit(raw, n_jobs=4)
```

### Logging

```python
# Set verbosity
mne.set_log_level('WARNING')  # or 'ERROR', 'INFO', 'DEBUG'
```

### Saving/Loading

```python
# Save in FIF format (recommended)
raw.save('processed-raw.fif', overwrite=True)
epochs.save('task-epo.fif', overwrite=True)
evoked.save('task-ave.fif', overwrite=True)

# Export to other formats
epochs.to_data_frame().to_csv('epochs.csv')
```
