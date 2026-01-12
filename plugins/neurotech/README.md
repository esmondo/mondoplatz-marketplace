# Neurotech Plugin for Claude Code

A comprehensive neuroscience, neurotechnology, and BCI knowledge base plugin for Claude Code.

## Features

- **MNE-Python guidance**: Complete reference for EEG/MEG analysis
- **BCI development**: P300, SSVEP, motor imagery paradigms
- **Signal processing**: Filtering, artifact removal, frequency analysis
- **Mental health biomarkers**: Depression, anxiety, ADHD markers
- **Market analysis**: Neurotech industry landscape

## Installation

1. Add the marketplace:
   ```
   /plugin marketplace add esmondo/neurotech
   ```

2. Install the plugin:
   ```
   /plugin install plugin-neurotech@mondo
   ```

3. Restart Claude Code

4. Check it worked:
   ```
   /plugin
   ```

## Usage

### Automatic Skill Activation

The neurotech skill triggers automatically when you mention:
- EEG, MEG, BCI, brain signals
- MNE, epochs, evoked, preprocessing
- Motor imagery, P300, SSVEP
- Neurofeedback, neural, biosignals

### Slash Command

Use `/plugin-neurotech:analyze-eeg` to get help with EEG/MEG analysis pipelines.

## What's Included

```
neurotech/
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
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

---

Built by Mondo for neuroscience and BCI developers.
