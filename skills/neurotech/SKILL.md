---
name: neurotech
description: Comprehensive neuroscience, neurotechnology, and brain-computer interface (BCI) knowledge base. Use when working with EEG/MEG/fMRI/EMG signal processing, BCI systems (P300, SSVEP, motor imagery), MNE-Python analysis, mental health biomarkers, neurotech product development, or neural data pipelines. Triggers on terms like EEG, MEG, BCI, brain signals, neural, MNE, epochs, evoked, preprocessing, artifacts, frequency bands, motor imagery, P300, neurofeedback.
---

# Neurotech Skill

Support neuroscience, neurotechnology, and BCI work with implementation guidance and evidence-based references.

## Reference Loading

Load only the reference needed for the current task.

| Task Type | Reference File |
|-----------|----------------|
| MNE-Python code, epochs, evoked, preprocessing | `references/mne-python.md` |
| BCI paradigms, P300, SSVEP, motor imagery, hardware | `references/bci-systems.md` |
| Filtering, artifacts, spectral analysis | `references/signal-processing.md` |
| Depression, anxiety, ADHD biomarkers | `references/mental-health-applications.md` |
| Brain anatomy, neural oscillations | `references/neuroscience-fundamentals.md` |
| Market analysis, regulatory, product dev | `references/neurotech-market.md` |
| Scientific validation, clinical evidence | `references/evidence-based-validation.md` |
| Journals, courses, datasets | `references/learning-resources.md` |
| Python/MATLAB packages, APIs | `references/tools-and-packages.md` |

## Search Patterns for Large Files

Use grep to find specific sections without loading entire files:

```bash
# BCI paradigms
grep -n "Motor Imagery\|P300\|SSVEP\|oddball\|CSP" references/bci-systems.md

# Signal processing
grep -n "bandpass\|notch\|ICA\|artifact\|PSD\|Welch" references/signal-processing.md

# Mental health markers
grep -n "depression\|anxiety\|alpha asymmetry\|theta/beta" references/mental-health-applications.md

# MNE functions
grep -n "Raw\|Epochs\|Evoked\|read_raw\|filter\|pick" references/mne-python.md
```

## Quick Reference

### EEG Frequency Bands
| Band | Range | Associated State |
|------|-------|------------------|
| Delta | 0.5-4 Hz | Deep sleep |
| Theta | 4-8 Hz | Drowsiness, meditation |
| Alpha | 8-13 Hz | Relaxed, eyes closed |
| Beta | 13-30 Hz | Active thinking |
| Gamma | 30-100 Hz | Higher cognition |

### MNE-Python Core Objects
```python
# Data containers
mne.io.Raw          # Continuous data
mne.Epochs          # Segmented trials
mne.Evoked          # Averaged responses
mne.SourceEstimate  # Source-space data

# Common workflow
raw = mne.io.read_raw_edf('data.edf', preload=True)
raw.filter(1, 40)
events = mne.find_events(raw)
epochs = mne.Epochs(raw, events, tmin=-0.2, tmax=0.5)
evoked = epochs.average()
```

### Standard Preprocessing Pipeline
1. Load raw data
2. Set montage (channel locations)
3. Filter (0.5-50 Hz bandpass, 50/60 Hz notch)
4. Re-reference (average or linked mastoids)
5. ICA for eye/muscle artifact removal
6. Epoch around events
7. Baseline correction
8. Reject bad epochs

## Response Guidelines

- Provide Python code with MNE-Python as primary library
- Include brief explanations of neuroscience concepts in code comments
- Reference specific functions and their parameters
- Acknowledge signal quality limitations
- Suggest validation approaches for claims
