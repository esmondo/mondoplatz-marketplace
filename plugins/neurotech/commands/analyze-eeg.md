---
description: Get help with EEG/MEG analysis using MNE-Python. Use this when starting a new analysis or troubleshooting preprocessing pipelines.
---

# EEG Analysis Helper

Help the user with their EEG or MEG analysis task. 

First, understand their goal:
- What data format do they have? (EDF, BDF, BrainVision, EEGLAB, etc.)
- What is their research question? (ERP, frequency analysis, BCI, source localization)
- What preprocessing have they done or need to do?

Then provide:
1. A clear preprocessing pipeline recommendation
2. Working MNE-Python code with comments explaining each step
3. Visualization suggestions
4. Common pitfalls to avoid for their specific analysis

Always load the neurotech skill references for detailed guidance:
- `skills/neurotech/references/mne-python.md` for MNE code patterns
- `skills/neurotech/references/signal-processing.md` for preprocessing theory
- `skills/neurotech/references/bci-systems.md` if doing BCI work
