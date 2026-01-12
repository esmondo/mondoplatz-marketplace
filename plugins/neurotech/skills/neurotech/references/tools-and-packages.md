# Tools and Packages

## Python Ecosystem

### MNE-Python (Primary EEG/MEG Library)

**Installation**:
```bash
pip install mne --break-system-packages
```

**Core Operations**:
```python
import mne
import numpy as np

# Load data
raw = mne.io.read_raw_edf('data.edf', preload=True)

# Preprocessing
raw.filter(l_freq=1.0, h_freq=40.0)
raw.set_eeg_reference('average')

# ICA artifact removal
from mne.preprocessing import ICA
ica = ICA(n_components=20, random_state=97)
ica.fit(raw)
ica.exclude = [0, 1]  # Exclude components
raw_clean = ica.apply(raw.copy())

# Epoching
events = mne.find_events(raw)
epochs = mne.Epochs(raw, events, tmin=-0.2, tmax=1.0)

# Time-frequency analysis
from mne.time_frequency import tfr_morlet
freqs = np.arange(4, 40, 2)
power = tfr_morlet(epochs, freqs=freqs, n_cycles=freqs/2)
power.plot([0])
```

### SciPy (Signal Processing)

```python
from scipy.signal import butter, filtfilt, welch, spectrogram

# Filtering
b, a = butter(4, [8, 13], btype='band', fs=256)
filtered = filtfilt(b, a, signal)

# Power spectrum
freqs, psd = welch(signal, fs=256, nperseg=256)

# Spectrogram
f, t, Sxx = spectrogram(signal, fs=256)
```

### scikit-learn (Machine Learning)

```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.svm import SVC
from sklearn.model_selection import cross_val_score

# Train classifier
lda = LinearDiscriminantAnalysis()
lda.fit(X_train, y_train)
accuracy = lda.score(X_test, y_test)

# Cross-validation
scores = cross_val_score(lda, X, y, cv=5)
```

### PyRiemann (Riemannian Geometry for BCI)

```python
from pyriemann.estimation import Covariances
from pyriemann.classification import MDM

# Estimate covariance matrices
cov = Covariances().fit_transform(X_train)

# Classify using minimum distance to mean
mdm = MDM()
mdm.fit(cov, y_train)
```

### Lab Streaming Layer (Real-Time Data)

```python
import pylsl

# Find and connect to stream
streams = pylsl.resolve_stream('type', 'EEG')
inlet = pylsl.StreamInlet(streams[0])

# Pull samples
while True:
    sample, timestamp = inlet.pull_sample()
    process_sample(sample)
```

### Hardware-Specific APIs

**Muse Headband**:
```python
# Using muselsl
from muselsl import stream, list_muses

# List available Muses
muses = list_muses()

# Start streaming
stream(muses[0]['address'])

# Then use pylsl to receive data
```

**OpenBCI**:
```python
from brainflow import BoardShim, BrainFlowInputParams, BoardIds

params = BrainFlowInputParams()
params.serial_port = 'COM3'  # or /dev/ttyUSB0 on Linux
board = BoardShim(BoardIds.CYTON_BOARD, params)

board.prepare_session()
board.start_stream()
data = board.get_board_data()
board.stop_stream()
board.release_session()
```

**Emotiv**:
```python
# Using cortex API (requires Emotiv account)
from cortex import Cortex

cortex = Cortex('./cortex_credentials')
cortex.do_prepare_steps()
cortex.subscribe(streams=['eeg'])
```

## MATLAB Ecosystem

### EEGLAB

```matlab
% Start EEGLAB
eeglab

% Load data
EEG = pop_loadset('data.set');

% Filter
EEG = pop_eegfiltnew(EEG, 1, 40);

% Re-reference
EEG = pop_reref(EEG, []);

% ICA
EEG = pop_runica(EEG, 'icatype', 'runica');

% Remove components
EEG = pop_subcomp(EEG, [1 2], 0);

% Epoch
EEG = pop_epoch(EEG, {'stim'}, [-0.2 1.0]);

% Plot ERP
figure; pop_timtopo(EEG, [-200 1000]);
```

### FieldTrip

```matlab
% Load data
cfg = [];
cfg.dataset = 'data.edf';
data = ft_preprocessing(cfg);

% Filter
cfg = [];
cfg.bpfilter = 'yes';
cfg.bpfreq = [1 40];
data_filt = ft_preprocessing(cfg, data);

% Time-frequency analysis
cfg = [];
cfg.method = 'mtmconvol';
cfg.foi = 4:2:40;
cfg.t_ftimwin = ones(size(cfg.foi)) * 0.5;
cfg.toi = -0.5:0.05:1.5;
TFR = ft_freqanalysis(cfg, data);

% Plot
cfg = [];
ft_singleplotTFR(cfg, TFR);
```

### Brainstorm

```matlab
% Start Brainstorm
brainstorm

% Import raw data, process via GUI
% Excellent for source localization

% Script example
% [sFiles, iStudy] = bst_process('CallProcess', 'process_import_data_raw', [], []);
% sFiles = bst_process('CallProcess', 'process_bandpass', sFiles, [], 'highpass', 1, 'lowpass', 40);
```

## Common Analysis Patterns

### Band Power Extraction

**Python**:
```python
def extract_band_powers(signal, fs=256):
    from scipy.signal import welch
    
    freqs, psd = welch(signal, fs=fs, nperseg=min(256, len(signal)))
    
    bands = {
        'delta': (0.5, 4),
        'theta': (4, 8),
        'alpha': (8, 13),
        'beta': (13, 30),
        'gamma': (30, 50)
    }
    
    powers = {}
    for band, (low, high) in bands.items():
        idx = np.logical_and(freqs >= low, freqs <= high)
        powers[band] = np.trapz(psd[idx], freqs[idx])
    
    return powers
```

**MATLAB**:
```matlab
function powers = extractBandPowers(signal, fs)
    [psd, freqs] = pwelch(signal, [], [], [], fs);
    
    bands = struct('delta', [0.5 4], 'theta', [4 8], ...
                   'alpha', [8 13], 'beta', [13 30], 'gamma', [30 50]);
    
    powers = struct();
    bandNames = fieldnames(bands);
    for i = 1:length(bandNames)
        band = bandNames{i};
        range = bands.(band);
        idx = freqs >= range(1) & freqs <= range(2);
        powers.(band) = trapz(freqs(idx), psd(idx));
    end
end
```

### Real-Time BCI Pipeline

```python
class SimpleBCI:
    def __init__(self, model_path):
        import joblib
        self.clf = joblib.load(model_path)
        self.buffer = np.zeros(256)  # 1 second at 256 Hz
        self.idx = 0
        
    def add_sample(self, sample):
        self.buffer[self.idx] = sample
        self.idx = (self.idx + 1) % len(self.buffer)
        
        if self.idx == 0:  # Buffer full
            return self.classify()
        return None
    
    def classify(self):
        features = self.extract_features(self.buffer)
        prediction = self.clf.predict([features])[0]
        confidence = self.clf.predict_proba([features])[0]
        return prediction, confidence
    
    def extract_features(self, signal):
        # Band powers as features
        powers = extract_band_powers(signal)
        return list(powers.values())

# Usage
bci = SimpleBCI('trained_model.pkl')

while True:
    sample = get_eeg_sample()  # From hardware
    result = bci.add_sample(sample)
    if result:
        prediction, confidence = result
        if confidence[prediction] > 0.7:
            execute_command(prediction)
```

## Visualization

### Python

```python
import matplotlib.pyplot as plt
import seaborn as sns

# Time series plot
plt.figure(figsize=(12, 4))
plt.plot(times, signal)
plt.xlabel('Time (s)')
plt.ylabel('Amplitude (μV)')
plt.title('EEG Signal')

# Power spectrum
plt.figure(figsize=(10, 6))
plt.semilogy(freqs, psd)
plt.xlabel('Frequency (Hz)')
plt.ylabel('Power (μV²/Hz)')
plt.title('Power Spectral Density')

# Topomap (requires MNE)
mne.viz.plot_topomap(data, epochs.info, show=True)

# Time-frequency
plt.pcolormesh(times, frequencies, power, shading='gouraud')
plt.colorbar(label='Power')
plt.ylabel('Frequency (Hz)')
plt.xlabel('Time (s)')
```

### MATLAB

```matlab
% Time series
figure;
plot(times, signal);
xlabel('Time (s)');
ylabel('Amplitude (\muV)');
title('EEG Signal');

% Power spectrum
figure;
semilogy(freqs, psd);
xlabel('Frequency (Hz)');
ylabel('Power (\muV^2/Hz)');
title('Power Spectral Density');

% Topomap (EEGLAB)
figure; topoplot(data, EEG.chanlocs);

% Time-frequency
figure;
imagesc(times, frequencies, power);
set(gca, 'YDir', 'normal');
colorbar;
ylabel('Frequency (Hz)');
xlabel('Time (s)');
```

## Deep Learning for EEG

### EEGNet (Compact CNN for EEG)

```python
from tensorflow.keras.models import Model
from tensorflow.keras.layers import Dense, Activation, Permute, Dropout
from tensorflow.keras.layers import Conv2D, MaxPooling2D, AveragePooling2D
from tensorflow.keras.layers import SeparableConv2D, DepthwiseConv2D
from tensorflow.keras.layers import BatchNormalization
from tensorflow.keras.layers import SpatialDropout2D
from tensorflow.keras.layers import Input, Flatten

def EEGNet(nb_classes, Chans=64, Samples=128, 
           dropoutRate=0.5, kernLength=64, F1=8, D=2, F2=16):
    
    input1 = Input(shape=(Chans, Samples, 1))
    
    # Block 1
    block1 = Conv2D(F1, (1, kernLength), padding='same', use_bias=False)(input1)
    block1 = BatchNormalization()(block1)
    block1 = DepthwiseConv2D((Chans, 1), use_bias=False, depth_multiplier=D)(block1)
    block1 = BatchNormalization()(block1)
    block1 = Activation('elu')(block1)
    block1 = AveragePooling2D((1, 4))(block1)
    block1 = Dropout(dropoutRate)(block1)
    
    # Block 2
    block2 = SeparableConv2D(F2, (1, 16), padding='same', use_bias=False)(block1)
    block2 = BatchNormalization()(block2)
    block2 = Activation('elu')(block2)
    block2 = AveragePooling2D((1, 8))(block2)
    block2 = Dropout(dropoutRate)(block2)
    
    # Classifier
    flatten = Flatten()(block2)
    dense = Dense(nb_classes)(flatten)
    softmax = Activation('softmax')(dense)
    
    return Model(inputs=input1, outputs=softmax)

# Usage
model = EEGNet(nb_classes=4, Chans=8, Samples=256)
model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])
model.fit(X_train, y_train, epochs=300, batch_size=16, validation_split=0.2)
```

## Data Management

### Saving and Loading

**Python (NumPy/Pickle)**:
```python
import numpy as np
import pickle

# Save arrays
np.save('eeg_data.npy', data)

# Load arrays
data = np.load('eeg_data.npy')

# Save objects (models, preprocessors)
with open('model.pkl', 'wb') as f:
    pickle.dump(model, f)

# Load
with open('model.pkl', 'rb') as f:
    model = pickle.load(f)
```

**Python (MNE formats)**:
```python
# Save raw data
raw.save('processed_raw.fif', overwrite=True)

# Save epochs
epochs.save('epochs-epo.fif', overwrite=True)

# Load
raw = mne.io.read_raw_fif('processed_raw.fif', preload=True)
epochs = mne.read_epochs('epochs-epo.fif')
```

**MATLAB**:
```matlab
% Save
save('eeg_data.mat', 'data', 'fs', 'channels');

% Load
load('eeg_data.mat');

% EEGLAB format
pop_saveset(EEG, 'filename', 'processed.set');
EEG = pop_loadset('processed.set');
```

## Performance Optimization

### Parallel Processing (Python)

```python
from joblib import Parallel, delayed

# Process multiple subjects in parallel
results = Parallel(n_jobs=4)(
    delayed(process_subject)(subject_id) 
    for subject_id in subjects
)

# Parallel cross-validation
from sklearn.model_selection import cross_val_score
scores = cross_val_score(clf, X, y, cv=5, n_jobs=-1)
```

### GPU Acceleration

```python
# TensorFlow/Keras automatically uses GPU if available
import tensorflow as tf
print("GPUs Available: ", len(tf.config.list_physical_devices('GPU')))

# For custom operations
import cupy as cp  # GPU-accelerated NumPy
gpu_array = cp.array(cpu_array)
result = cp.fft.fft(gpu_array)
cpu_result = cp.asnumpy(result)
```

## Useful Utilities

### Event Markers

```python
def create_events(timestamps, event_codes):
    """Create MNE events array."""
    events = np.column_stack([
        timestamps,
        np.zeros(len(timestamps), dtype=int),
        event_codes
    ])
    return events

# Usage
events = create_events([100, 250, 400], [1, 2, 1])
```

### Channel Selection

```python
# Select specific channels
picks = mne.pick_types(raw.info, eeg=True, exclude='bads')
motor_picks = mne.pick_channels(raw.ch_names, ['C3', 'Cz', 'C4'])

# Get data from selected channels
data = raw.get_data(picks=motor_picks)
```

### Artifact Rejection

```python
# Amplitude-based rejection
reject_criteria = dict(eeg=100e-6)  # 100 μV
epochs.drop_bad(reject=reject_criteria)

# Manual inspection
epochs.plot(n_epochs=5, n_channels=30, scalings='auto')
# Mark bad epochs interactively
```

## Troubleshooting

### Common Issues

**"Filter length exceeds data length"**:
- Solution: Use shorter filter or more data
```python
raw.filter(l_freq=1.0, h_freq=40.0, fir_design='firwin', 
           filter_length='auto', l_trans_bandwidth='auto')
```

**"Channels not found"**:
- Check channel names match exactly (case-sensitive)
- Use `raw.ch_names` to see available channels

**"Memory error"**:
- Process data in chunks
- Use `preload=False` and load incrementally
- Reduce sampling rate if appropriate

**"ICA doesn't converge"**:
- Increase `max_iter` parameter
- Filter data first (0.5-40 Hz)
- Check for bad channels

## Further Resources

- MNE tutorials: https://mne.tools/stable/auto_tutorials/
- EEGLAB tutorial: https://eeglab.org/tutorials/
- PyRiemann docs: https://pyriemann.readthedocs.io/
- BrainFlow docs: https://brainflow.readthedocs.io/

For hardware-specific APIs:
- Muse: https://github.com/alexandrebarachant/muse-lsl
- OpenBCI: https://docs.openbci.com/
- Emotiv: https://emotiv.gitbook.io/cortex-api/
