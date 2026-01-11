# BCI Systems

## Overview

Brain-Computer Interfaces (BCIs) enable direct communication between the brain and external devices. This guide covers BCI paradigms, hardware considerations, real-time processing pipelines, and machine learning approaches for BCI development.

## BCI Classification

### By Invasiveness

**Invasive BCIs**:
- Electrodes implanted directly in brain tissue
- Highest signal quality and spatial resolution
- Requires neurosurgery, risk of infection/rejection
- Examples: Utah array, Neuralink, BrainGate
- Use case: Severe paralysis, research

**Partially Invasive (ECoG)**:
- Electrodes on brain surface (under skull, above brain)
- Better signal than EEG, less invasive than penetrating
- Still requires surgery
- Use case: Epilepsy monitoring, research

**Non-Invasive BCIs**:
- Sensors on scalp surface (EEG most common)
- Lower signal quality, convenient, no surgery
- Lower cost, easier to use
- Examples: Consumer EEG headsets, research caps
- Use case: Consumer applications, research, clinical

### By Control Signal

**Endogenous (Self-Paced)**:
- User generates signal voluntarily
- No external stimulus required
- Examples: Motor imagery, mental tasks
- More challenging but more flexible

**Exogenous (Stimulus-Driven)**:
- Brain response to external stimuli
- Requires presenting stimuli
- Examples: P300, SSVEP
- Generally easier to achieve good performance

**Hybrid BCIs**:
- Combine multiple control signals or modalities
- Improve accuracy and reliability
- Example: P300 + motor imagery

## Major BCI Paradigms

### Motor Imagery (MI)

**Concept**: Imagine movement without actual movement.

**Neural Basis**:
- Engages motor cortex similar to actual movement
- Event-related desynchronization (ERD) in mu (8-12 Hz) and beta (13-30 Hz) bands
- Contralateral motor cortex shows stronger ERD
- Can distinguish left hand, right hand, feet, tongue imagery

**Recording Setup**:
- Focus on motor cortex: C3, Cz, C4 electrodes
- Minimum 3 channels, more is better
- Sampling rate: 250 Hz+
- Session: Multiple trials (20-40 per class)

**Processing Pipeline**:
1. Bandpass filter 8-30 Hz (mu and beta bands)
2. Epoch around imagery period (e.g., 0-4 seconds)
3. Extract features (band power, CSP features)
4. Train classifier (LDA, SVM)
5. Real-time classification of new imagery

```python
import mne
from mne.decoding import CSP
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.pipeline import Pipeline

# Load and preprocess data
epochs = mne.read_epochs('motor_imagery_epochs.fif')

# Define classes (left hand vs right hand)
X = epochs.get_data()
y = epochs.events[:, 2]

# Create processing pipeline
csp = CSP(n_components=4, reg=None, log=True)
lda = LinearDiscriminantAnalysis()
clf = Pipeline([('CSP', csp), ('LDA', lda)])

# Train classifier
clf.fit(X, y)

# Predict new trials
predictions = clf.predict(X_test)
accuracy = clf.score(X_test, y_test)
print(f"Classification accuracy: {accuracy:.2%}")
```

**Training Considerations**:
- Users need practice (10-30 minutes minimum)
- Individual differences are large
- Feedback improves performance
- Fatigue affects performance

**Applications**:
- Wheelchair control
- Robotic arm control
- Spelling systems
- Gaming

### P300 (Oddball Paradigm)

**Concept**: Rare, task-relevant stimuli evoke positive deflection at ~300 ms.

**Neural Basis**:
- P300 component: Large positive potential
- Parietal electrodes show strongest response
- Reflects attention and memory updating
- Reliable, well-characterized ERP

**Recording Setup**:
- Focus on parietal: Pz, P3, P4, Cz
- Minimum 4 channels
- Sampling rate: 256 Hz+
- Present stimuli in rapid succession (ISI: 250-500 ms)

**Classic Example - P300 Speller**:
- 6×6 matrix of characters
- Flash rows and columns randomly
- User attends to target character
- P300 occurs when target row/column flashes
- Detect which row and column evoked P300

```python
import numpy as np
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

# Prepare data
epochs_target = epochs['target']  # Target flashes
epochs_nontarget = epochs['non_target']  # Non-target flashes

# Extract features (mean amplitude 250-450 ms post-stimulus)
def extract_p300_features(epochs):
    data = epochs.copy().crop(0.25, 0.45).average().data
    return data.mean(axis=1)  # Average across time

X_target = np.array([extract_p300_features(epochs_target[i:i+1]) 
                     for i in range(len(epochs_target))])
X_nontarget = np.array([extract_p300_features(epochs_nontarget[i:i+1]) 
                        for i in range(len(epochs_nontarget))])

X = np.vstack([X_target, X_nontarget])
y = np.hstack([np.ones(len(X_target)), np.zeros(len(X_nontarget))])

# Train classifier
clf = LogisticRegression(max_iter=1000)
scores = cross_val_score(clf, X, y, cv=5)
print(f"Average accuracy: {scores.mean():.2%}")

# Online spelling: determine target character
clf.fit(X, y)
# For each row/column, average predictions across flashes
# Select row and column with highest probability
```

**Advantages**:
- Minimal training required
- High accuracy possible
- Robust across users
- Well-understood

**Limitations**:
- Requires external stimuli (cannot use freely)
- Relatively slow (6-12 selections/minute)
- Attention-demanding

**Applications**:
- Communication devices for locked-in patients
- Spelling systems
- Menu selection
- Yes/no detection

### Steady-State Visual Evoked Potentials (SSVEP)

**Concept**: Flickering visual stimuli evoke oscillations at flicker frequency.

**Neural Basis**:
- Visual cortex entrains to stimulus frequency
- Response at fundamental frequency and harmonics
- Occipital electrodes show strongest response
- Phase-locked to stimulus

**Recording Setup**:
- Focus on occipital: O1, Oz, O2
- Minimum 2-3 channels
- Sampling rate: 256 Hz+ (higher better)
- Multiple stimuli flickering at different frequencies (e.g., 6, 7, 8, 9 Hz)

**Processing**:
1. Bandpass filter around stimulus frequencies
2. Compute power spectrum or canonical correlation analysis (CCA)
3. Detect which frequency has strongest response
4. Map frequency to command

```python
from scipy.signal import welch, coherence
from sklearn.cross_decomposition import CCA

# Method 1: Power spectrum
frequencies, psd = welch(signal, fs=256, nperseg=256)
# Find peak at stimulus frequency
stimulus_freq = 8.0  # Hz
peak_power = psd[np.argmin(np.abs(frequencies - stimulus_freq))]

# Method 2: CCA (more robust)
def ssvep_cca(signal, fs, stimulus_freq, n_harmonics=2):
    """Detect SSVEP using Canonical Correlation Analysis."""
    # Create reference signals at stimulus frequency and harmonics
    duration = len(signal) / fs
    t = np.linspace(0, duration, len(signal))
    
    Y = []
    for h in range(1, n_harmonics + 1):
        Y.append(np.sin(2 * np.pi * h * stimulus_freq * t))
        Y.append(np.cos(2 * np.pi * h * stimulus_freq * t))
    Y = np.array(Y).T
    
    # CCA between signal and reference
    cca = CCA(n_components=1)
    X_c, Y_c = cca.fit_transform(signal.reshape(-1, 1), Y)
    
    # Correlation as detection metric
    corr = np.corrcoef(X_c.T, Y_c.T)[0, 1]
    return corr

# Test multiple stimulus frequencies
stimulus_frequencies = [6.0, 7.0, 8.0, 9.0, 10.0]
correlations = [ssvep_cca(signal, 256, freq) for freq in stimulus_frequencies]
detected_freq = stimulus_frequencies[np.argmax(correlations)]
print(f"Detected stimulus: {detected_freq} Hz")
```

**Advantages**:
- High information transfer rate (30-60 selections/minute)
- Minimal training
- High accuracy
- Robust across users

**Limitations**:
- Requires gaze shifting (fatiguing)
- May cause visual discomfort
- Limited number of targets (typically 4-40)
- Not suitable for visually impaired

**Applications**:
- High-speed spelling
- Device control
- Wheelchair navigation
- Gaming

### Slow Cortical Potentials (SCP)

**Concept**: Voluntary control of slow voltage shifts (<1 Hz).

**Neural Basis**:
- Slow DC shifts in cortical excitability
- Can be learned with extensive training
- Reflects cortical preparation and attention

**Processing**:
- Very low frequency: 0.1-1 Hz
- Requires DC-coupled amplifier or very low high-pass filter
- Average amplitude over time windows (1-2 seconds)

**Limitations**:
- Long training required (weeks to months)
- Only two states (negative/positive)
- Low information transfer rate
- Less commonly used today

**Applications**:
- Binary selection (yes/no, up/down)
- Cursor control

### Error-Related Potentials (ErrP)

**Concept**: Detect when user recognizes BCI made an error.

**Neural Basis**:
- Error-related negativity (ERN) at ~50-100 ms
- Error positivity (Pe) at ~200-400 ms
- Frontocentral scalp distribution

**Application**:
- Improve BCI reliability by detecting errors
- Automatic error correction
- Combined with other BCI paradigms

### Hybrid BCIs

**Rationale**: Combining signals improves performance and usability.

**Common Combinations**:
- **P300 + SSVEP**: Use P300 for selection, SSVEP for navigation
- **MI + P300**: MI for continuous control, P300 for discrete commands
- **EEG + EMG**: Brain signals + muscle signals for robust control
- **EEG + Eye tracking**: Combine gaze with neural commands

**Advantages**:
- Higher accuracy
- More degrees of freedom
- Redundancy improves reliability
- Flexibility in control strategies

## Hardware Considerations

### Electrode Placement

**International 10-20 System**:
- Standard for electrode positioning
- Based on skull measurements
- Ensures reproducibility across subjects and studies

**Key Locations by Paradigm**:
- **Motor Imagery**: C3, Cz, C4 (motor cortex)
- **P300**: Pz, P3, P4, Cz (parietal)
- **SSVEP**: O1, Oz, O2 (occipital)
- **General**: Fz (reference), ground on forehead

**Montage Design**:
- More channels generally better but diminishing returns
- Consumer: 2-8 channels, focus on region of interest
- Research: 32-256 channels for source localization

### Electrode Types

**Wet Electrodes**:
- Require conductive gel
- Best signal quality (low impedance)
- Time-consuming to apply
- Uncomfortable for long periods
- Standard for research

**Dry Electrodes**:
- No gel required
- Faster setup
- More comfortable for long use
- Slightly worse signal quality
- Better for consumer applications

**Active vs Passive**:
- **Passive**: Simple metal contacts
- **Active**: Built-in amplification, better noise rejection

### Sampling Rate

**Minimum Requirements by Paradigm**:
- **Motor Imagery**: 128 Hz (250 Hz recommended)
- **P300**: 256 Hz
- **SSVEP**: 256 Hz+ (higher better, 512 Hz+ for high-frequency SSVEP)
- **Research ERP**: 500-1000 Hz

**Trade-offs**:
- Higher sampling: Better temporal resolution, larger data files
- Nyquist theorem: Must sample at ≥2× highest frequency of interest

### Signal Quality Metrics

**Impedance**:
- Measure of electrode-skin contact resistance
- Target: <5 kΩ (research), <10 kΩ (consumer acceptable)
- Check before and during recording

**Signal-to-Noise Ratio (SNR)**:
- Ratio of signal power to noise power
- Higher is better
- Varies by paradigm: SSVEP typically high, MI moderate

**Electrode Drift**:
- Signal baseline shifts over time
- More common with dry electrodes
- Mitigate with high-pass filtering, regular recalibration

### Consumer EEG Devices

**Muse (Interaxon)**:
- 4-7 electrodes (depending on model)
- Dry electrodes, comfortable
- Bluetooth connectivity
- Good for meditation, attention monitoring
- Python SDK available (muselsl)

**Emotiv EPOC/Insight**:
- 5-14 channels
- Saline-based semi-dry electrodes
- Wireless, good battery life
- Emotiv SDK for development

**OpenBCI**:
- 8-16 channels (expandable)
- Wet or dry electrodes
- Open-source hardware/software
- Flexible for custom applications
- Arduino-compatible

**NeuroSky MindWave**:
- Single channel (Fp1)
- Very limited but low cost
- Good for simple attention/relaxation metrics

**Selection Criteria**:
- Number of channels (more = better spatial resolution)
- Electrode quality and comfort
- Sampling rate
- Software ecosystem and API
- Price vs. performance
- Target application (research vs. consumer)

## Real-Time Processing Pipeline

### Architecture

**Components**:
1. **Data acquisition**: Streaming from hardware
2. **Preprocessing**: Filtering, artifact removal
3. **Feature extraction**: Convert signals to features
4. **Classification**: Predict user intent
5. **Feedback**: Provide results to user and system

**Latency Requirements**:
- Total latency: <300 ms ideal, <500 ms acceptable
- Feedback delay affects performance and user experience

### Streaming Data

**Buffer Management**:
```python
import pylsl  # Lab Streaming Layer

# Connect to EEG stream
streams = pylsl.resolve_stream('type', 'EEG')
inlet = pylsl.StreamInlet(streams[0])

# Circular buffer for continuous processing
buffer_size = 1024  # 4 seconds at 256 Hz
buffer = np.zeros(buffer_size)
buffer_idx = 0

while True:
    # Pull sample from stream
    sample, timestamp = inlet.pull_sample()
    
    # Add to buffer
    buffer[buffer_idx] = sample[0]  # First channel
    buffer_idx = (buffer_idx + 1) % buffer_size
    
    # Process when buffer full or at regular intervals
    if buffer_idx == 0:
        process_buffer(buffer)
```

### Online Filtering

**Considerations**:
- Causal filters only (no future data)
- Minimize phase lag
- Use IIR filters for efficiency

```python
from scipy.signal import butter, lfilter

# Design filter once
b, a = butter(4, [8, 30], btype='band', fs=256)

# Initialize filter state for continuous filtering
zi = np.zeros(max(len(a), len(b)) - 1)

# Filter each incoming sample
def filter_sample(sample):
    global zi
    filtered, zi = lfilter(b, a, [sample], zi=zi)
    return filtered[0]
```

### Feature Extraction

**Windowing**:
- Extract features from recent time window
- Typical: 1-4 seconds
- Sliding window with overlap

**Common Features**:
- **Band power**: Power in specific frequency bands
- **Peak amplitude**: Maximum or minimum in time window
- **Autoregressive (AR) coefficients**: Model signal as autoregressive process
- **Wavelet features**: Time-frequency information
- **Hjorth parameters**: Activity, mobility, complexity

```python
def extract_band_power_features(signal, fs, bands):
    """Extract power in multiple frequency bands."""
    from scipy.signal import welch
    
    freqs, psd = welch(signal, fs=fs, nperseg=min(256, len(signal)))
    
    features = []
    for low, high in bands:
        idx = np.logical_and(freqs >= low, freqs <= high)
        power = np.trapz(psd[idx], freqs[idx])
        features.append(power)
    
    return np.array(features)

# Use in real-time
bands = [(8, 12), (13, 30)]  # Alpha and beta
features = extract_band_power_features(buffer, fs=256, bands=bands)
```

### Classification

**Online Prediction**:
```python
# Load pre-trained classifier
import joblib
clf = joblib.load('trained_classifier.pkl')

# Predict from features
prediction = clf.predict(features.reshape(1, -1))[0]
confidence = clf.predict_proba(features.reshape(1, -1))[0]

# Apply threshold for robust detection
if confidence[prediction] > 0.7:
    execute_command(prediction)
```

**Adaptive Classifiers**:
- Update model with new data during use
- Compensate for signal drift
- Balance stability vs. adaptation

### Artifact Detection

**Real-Time Methods**:
- Amplitude threshold: Reject if |signal| > threshold
- Gradient threshold: Reject if rapid change
- Frequency content: Reject if high-frequency energy too high

```python
def detect_artifacts(signal, threshold_uv=100):
    """Simple amplitude-based artifact detection."""
    if np.max(np.abs(signal)) > threshold_uv:
        return True
    return False

# In processing loop
if detect_artifacts(buffer):
    # Skip this window or interpolate
    continue
```

## Machine Learning for BCI

### Data Preparation

**Balancing Classes**:
- Equal number of trials per class
- Oversample minority class or undersample majority

**Feature Scaling**:
- Standardize features (zero mean, unit variance)
- Important for many classifiers (LDA, SVM, neural networks)

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

**Cross-Validation**:
- Avoid overfitting
- K-fold cross-validation (typically 5-10 folds)
- Leave-one-session-out for temporal generalization

### Common Classifiers

**Linear Discriminant Analysis (LDA)**:
- Simple, fast, interpretable
- Assumes Gaussian distributions
- Works well with few features
- Common choice for BCI

```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis

lda = LinearDiscriminantAnalysis()
lda.fit(X_train, y_train)
accuracy = lda.score(X_test, y_test)
```

**Support Vector Machine (SVM)**:
- Powerful, flexible
- Works well with high-dimensional data
- Kernel trick for non-linear boundaries
- More computationally expensive

```python
from sklearn.svm import SVC

svm = SVC(kernel='rbf', gamma='auto')
svm.fit(X_train, y_train)
accuracy = svm.score(X_test, y_test)
```

**Neural Networks**:
- Can learn complex patterns
- Requires more data
- More computationally expensive
- EEGNet architecture designed for EEG

```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Dropout

model = Sequential([
    Dense(64, activation='relu', input_shape=(n_features,)),
    Dropout(0.5),
    Dense(32, activation='relu'),
    Dropout(0.5),
    Dense(n_classes, activation='softmax')
])

model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', 
              metrics=['accuracy'])
model.fit(X_train, y_train, validation_split=0.2, epochs=50, batch_size=32)
```

**Ensemble Methods**:
- Random forests, gradient boosting
- Often improve accuracy
- Slower for real-time use

### Specialized Techniques

**Common Spatial Patterns (CSP)**:
- Spatial filtering to maximize class separability
- Widely used for motor imagery
- Learns optimal electrode combinations

```python
from mne.decoding import CSP

csp = CSP(n_components=6, reg=None, log=True)
X_csp = csp.fit_transform(X_train, y_train)

# Visualize spatial patterns
csp.plot_patterns(epochs.info)
```

**Riemannian Geometry**:
- Operate on covariance matrices
- Robust to noise and artifacts
- Computationally efficient

```python
from pyriemann.classification import MDM
from pyriemann.estimation import Covariances

# Estimate covariance matrices
cov = Covariances().transform(X_train)
cov_test = Covariances().transform(X_test)

# Minimum distance to mean classifier
mdm = MDM()
mdm.fit(cov, y_train)
accuracy = mdm.score(cov_test, y_test)
```

## Performance Metrics

### Accuracy
- Percentage of correct classifications
- Baseline: 1/n_classes (chance level)
- Should be significantly above chance

### Information Transfer Rate (ITR)
- Bits per minute (or per trial)
- Accounts for accuracy, number of classes, and selection time
- Formula: ITR = log₂(N) + P·log₂(P) + (1-P)·log₂((1-P)/(N-1))
  - N = number of classes
  - P = accuracy

```python
import numpy as np

def calculate_itr(n_classes, accuracy, trial_duration):
    """Calculate Information Transfer Rate in bits/minute."""
    P = accuracy
    N = n_classes
    
    if P == 1:
        bits_per_trial = np.log2(N)
    elif P == 0:
        bits_per_trial = 0
    else:
        bits_per_trial = (np.log2(N) + P * np.log2(P) + 
                         (1 - P) * np.log2((1 - P) / (N - 1)))
    
    trials_per_minute = 60 / trial_duration
    itr = bits_per_trial * trials_per_minute
    
    return itr

# Example: 4-class problem, 75% accuracy, 5-second trials
itr = calculate_itr(4, 0.75, 5)
print(f"ITR: {itr:.2f} bits/minute")
```

### Confusion Matrix
- Shows which classes are confused
- Identifies specific problem areas

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay

cm = confusion_matrix(y_test, predictions)
disp = ConfusionMatrixDisplay(confusion_matrix=cm)
disp.plot()
```

## Practical Considerations

### Training Protocol

**Calibration Phase**:
- Collect labeled data for initial training
- Typically 50-200 trials (10-20 per class)
- 10-30 minutes duration
- Critical for good performance

**Instructions to Users**:
- Clear, specific task descriptions
- Avoid actual movement for MI
- Minimize blinking and movement
- Consistent attention and effort

**Feedback**:
- Provide real-time feedback during training
- Helps users learn to control BCI
- Visual, auditory, or haptic modalities

### User Variability

**BCI Illiteracy**:
- 10-30% of users cannot achieve good control
- More common with motor imagery
- Reasons unclear (motivation, attention, neural differences)

**Session-to-Session Variability**:
- Signal characteristics change across days
- Require recalibration or adaptive methods
- Transfer learning can reduce calibration time

**Fatigue**:
- Performance degrades over time (20-60 minutes)
- Take breaks between sessions
- Monitor performance, stop if degraded

### Ethical Considerations

**Informed Consent**:
- Explain capabilities and limitations
- Clarify data collection and use
- Allow withdrawal at any time

**Privacy**:
- Brain data is sensitive personal information
- Secure storage and transmission
- Limit access, anonymize when possible

**Safety**:
- Non-invasive BCIs generally safe
- Monitor for discomfort, seizures (for epileptic users)
- Avoid excessive stimulation (SSVEP)

## Further Reading

- See `signal-processing.md` for detailed preprocessing methods
- See `tools-and-packages.md` for BCI-specific libraries and examples
- See `neuroscience-fundamentals.md` for neural basis of BCI signals
- See `learning-resources.md` for BCI courses and textbooks
