# Neurotech Plugin for Claude Code

A comprehensive neuroscience, neurotechnology, and BCI knowledge base plugin for Claude Code.

## Features

- **MNE-Python guidance**: Complete reference for EEG/MEG analysis
- **BCI development**: P300, SSVEP, motor imagery paradigms
- **Signal processing**: Filtering, artifact removal, frequency analysis
- **Mental health biomarkers**: Depression, anxiety, ADHD markers
- **Market analysis**: Neurotech industry landscape

## Installation

### 1. Extract the zip file

```bash
unzip neurotech-marketplace.zip -d ~/
```

### 2. Add the marketplace to Claude Code

Start Claude Code and run:

```
/plugin marketplace add ~/neurotech-marketplace
```

### 3. Install the plugin

```
/plugin install neurotech@neurotech-marketplace
```

### 4. Restart Claude Code

The plugin is now active.

## Usage

### Automatic Skill Activation

The neurotech skill triggers automatically when you mention:
- EEG, MEG, BCI, brain signals
- MNE, epochs, evoked, preprocessing
- Motor imagery, P300, SSVEP
- Neurofeedback, neural, biosignals

### Slash Command

Use `/analyze-eeg` to get help with EEG/MEG analysis pipelines.

## What's Included

```
neurotech/
├── .claude-plugin/plugin.json
├── commands/
│   └── analyze-eeg.md
└── skills/neurotech/
    ├── SKILL.md
    └── references/
        ├── mne-python.md              # MNE-Python API reference
        ├── bci-systems.md             # BCI paradigms & hardware
        ├── signal-processing.md       # Preprocessing pipelines
        ├── mental-health-applications.md
        ├── neuroscience-fundamentals.md
        ├── neurotech-market.md
        ├── tools-and-packages.md
        ├── evidence-based-validation.md
        └── learning-resources.md
```

## Token Efficiency

The plugin uses progressive disclosure:
- Only SKILL.md loads on trigger (~2KB)
- Reference files load on-demand when needed
- Grep search patterns for finding specific sections in large files
