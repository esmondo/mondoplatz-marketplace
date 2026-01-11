# Mental Health Applications

## Overview

Neural monitoring and neurofeedback show promise for mental health applications. This guide covers neural markers, evidence base, and implementation considerations for depression, anxiety, ADHD, stress monitoring, and sleep management.

**Critical Note**: Always reference `evidence-based-validation.md` for current evidence status. Many commercial claims lack robust scientific support. See project Evidence Book for systematic review findings.

## Depression

### Neural Markers

**EEG Asymmetry**:
- **Frontal alpha asymmetry**: Reduced left frontal activity relative to right
- Measured as: log(right_alpha) - log(left_alpha)
- Electrodes: F3 (left), F4 (right)
- State marker: Can fluctuate with mood
- Trait marker: Persistent pattern in some individuals

**Oscillatory Changes**:
- **Increased delta and theta**: Associated with withdrawal, low arousal
- **Reduced alpha**: During active depression
- **Altered beta**: Variable findings across studies

**ERP Components**:
- **Reduced P300 amplitude**: Impaired attention, processing speed
- **Altered N200**: Conflict monitoring deficits
- **Error-related negativity (ERN)**: May be enhanced (worry, rumination)

**Connectivity**:
- **Reduced frontal-posterior connectivity**: Impaired cognitive control
- **Altered default mode network**: Excessive self-referential thinking
- **Decreased hemispheric connectivity**: Related to rumination

### Evidence Base

**What's Supported**:
- Frontal alpha asymmetry shows moderate group differences
- EEG can help predict treatment response in some patients
- Neurofeedback may reduce symptoms (but evidence is mixed)

**Limitations**:
- High individual variability
- Many studies have small samples or methodological issues
- Not diagnostic (cannot replace clinical assessment)
- Effect sizes often modest

**Clinical Validation Requirements**:
- Large-scale trials needed
- Comparison with standard treatments
- Long-term outcome monitoring
- Cost-effectiveness analysis

### Implementation Considerations

**Target Population**:
- Mild to moderate depression (not severe or psychotic)
- Adjunct to standard care, not replacement
- Consider motivation and compliance

**Neurofeedback Protocol**:
- **Target**: Increase left frontal alpha or reduce asymmetry
- **Duration**: 20-40 sessions typical
- **Frequency**: 2-3 times per week
- **Monitoring**: Track symptoms weekly (PHQ-9, BDI-II)

**Safety**:
- Monitor for worsening symptoms
- Screen for suicidality
- Have referral pathway for crisis
- Not appropriate for active suicidal ideation

```python
# Example: Calculate frontal alpha asymmetry
import numpy as np
from scipy.signal import welch

def frontal_alpha_asymmetry(eeg_left, eeg_right, fs=256):
    """
    Calculate frontal alpha asymmetry.
    
    Parameters:
    eeg_left: EEG signal from left frontal (e.g., F3)
    eeg_right: EEG signal from right frontal (e.g., F4)
    fs: Sampling frequency
    
    Returns:
    asymmetry: log(right_alpha) - log(left_alpha)
               Positive = greater right activity (depression pattern)
    """
    # Compute power spectral density
    freq_left, psd_left = welch(eeg_left, fs=fs, nperseg=256)
    freq_right, psd_right = welch(eeg_right, fs=fs, nperseg=256)
    
    # Extract alpha band (8-13 Hz)
    alpha_idx = np.logical_and(freq_left >= 8, freq_left <= 13)
    alpha_power_left = np.trapz(psd_left[alpha_idx], freq_left[alpha_idx])
    alpha_power_right = np.trapz(psd_right[alpha_idx], freq_right[alpha_idx])
    
    # Calculate asymmetry (log transform stabilizes variance)
    asymmetry = np.log(alpha_power_right) - np.log(alpha_power_left)
    
    return asymmetry

# Use in monitoring
asymmetry = frontal_alpha_asymmetry(f3_signal, f4_signal)
print(f"Frontal alpha asymmetry: {asymmetry:.3f}")
# Positive values suggest right-dominant pattern (associated with depression)
```

## Anxiety Disorders

### Neural Markers

**Oscillatory Patterns**:
- **Elevated beta (13-30 Hz)**: Hyperarousal, worry
- **Increased high-frequency activity**: Especially frontal regions
- **Reduced alpha**: Difficulty relaxing
- **Theta/beta ratio**: May be altered (less consistent than ADHD)

**ERP Components**:
- **Enhanced N200**: Heightened threat detection
- **Larger error-related negativity**: Excessive monitoring
- **Altered P300**: To threatening vs. neutral stimuli

**Heart Rate Variability (HRV)**:
- **Reduced HRV**: Lower parasympathetic activity
- **Respiratory sinus arrhythmia**: Reduced
- **Measure**: RMSSD, high-frequency HRV power
- Reflects autonomic dysregulation

**Connectivity**:
- **Heightened amygdala-frontal connectivity**: To threat cues
- **Reduced frontal inhibition**: Of limbic regions
- **Altered attention networks**: Bias toward threats

### Evidence Base

**What's Supported**:
- Beta activity correlates with anxiety symptoms in some studies
- HRV biofeedback has moderate evidence for anxiety reduction
- Neurofeedback may reduce symptoms (evidence growing but mixed)

**Limitations**:
- Overlap with normal arousal and attention
- Difficult to isolate anxiety-specific patterns
- Many anxiety subtypes (GAD, social, panic) may differ
- Individual variability high

### Implementation Considerations

**Target Population**:
- Generalized anxiety disorder (GAD)
- Performance anxiety
- Stress-related anxiety
- Complement to cognitive-behavioral therapy (CBT)

**Neurofeedback Protocol**:
- **Beta training**: Reduce beta power (especially 15-25 Hz)
- **Alpha training**: Increase alpha (relaxation)
- **HRV biofeedback**: Increase vagal tone (see stress section)
- **Duration**: 20-40 sessions
- **Monitoring**: Track anxiety (GAD-7, BAI) and physiological measures

**Safety**:
- Monitor for panic attacks during training
- Gradual progression
- Teach grounding techniques
- Have crisis plan

```python
# Example: Anxiety monitoring with beta/alpha ratio
def anxiety_index(eeg_signal, fs=256):
    """
    Calculate beta/alpha ratio as anxiety index.
    Higher values suggest increased anxiety/arousal.
    """
    from scipy.signal import welch
    
    freq, psd = welch(eeg_signal, fs=fs, nperseg=256)
    
    # Alpha (8-13 Hz) and Beta (13-30 Hz)
    alpha_idx = np.logical_and(freq >= 8, freq <= 13)
    beta_idx = np.logical_and(freq >= 13, freq <= 30)
    
    alpha_power = np.trapz(psd[alpha_idx], freq[alpha_idx])
    beta_power = np.trapz(psd[beta_idx], freq[beta_idx])
    
    ratio = beta_power / alpha_power
    return ratio

# Monitor over time
anxiety_score = anxiety_index(frontal_signal)
print(f"Beta/Alpha ratio: {anxiety_score:.2f}")
```

## ADHD (Attention-Deficit/Hyperactivity Disorder)

### Neural Markers

**Theta/Beta Ratio**:
- **Classic marker**: Elevated theta (4-8 Hz) / reduced beta (13-30 Hz)
- Measured at frontal-central sites (Fz, Cz)
- Reflects cortical hypoarousal
- Individual variability is high

**Slow Wave Activity**:
- **Increased delta and theta**: Particularly during attention tasks
- Reflects underactive prefrontal cortex
- Correlates with inattention symptoms

**ERP Components**:
- **Reduced P300 amplitude**: Impaired attention, inhibition
- **Delayed P300 latency**: Slower processing
- **Smaller contingent negative variation (CNV)**: Reduced preparation

**Oscillatory Patterns**:
- **Reduced beta during attention**: Insufficient arousal
- **Excessive slow oscillations**: Drowsy-like state while awake

### Evidence Base

**What's Supported**:
- Theta/beta ratio elevated in many (but not all) ADHD individuals
- Neurofeedback for ADHD has moderate evidence (some well-controlled studies)
- Can reduce symptoms, improve attention in responders

**Limitations**:
- Theta/beta ratio not diagnostic (overlap with typical development)
- Effects may be partly placebo or non-specific
- Long-term outcomes unclear
- Many studies lack rigorous controls

**FDA Status**:
- Some EEG-based devices FDA-cleared to aid ADHD diagnosis
- Not standalone diagnostic tools

### Implementation Considerations

**Target Population**:
- Children and adults with ADHD diagnosis
- Primarily inattentive or combined subtype
- Adjunct to medication and behavioral therapy
- Best for motivated individuals

**Neurofeedback Protocol**:
- **Theta/beta training**: Reduce theta, increase beta
- **Sensorimotor rhythm (SMR, 12-15 Hz)**: Increase for focus
- **Duration**: 30-50 sessions common
- **Frequency**: 2-3 times per week
- **Monitoring**: ADHD rating scales, objective attention tests

**Age Considerations**:
- Children (6+): Need engaging interface, shorter sessions
- Adolescents: Motivation can be challenging
- Adults: May have comorbidities to address

**Safety**:
- Generally safe
- Monitor for over-arousal (if beta increased too much)
- Watch for increased anxiety or sleep issues

```python
# Example: ADHD marker - theta/beta ratio
def theta_beta_ratio(eeg_signal, fs=256):
    """
    Calculate theta/beta ratio, a common ADHD marker.
    Higher values associated with ADHD.
    """
    from scipy.signal import welch
    
    freq, psd = welch(eeg_signal, fs=fs, nperseg=256)
    
    # Theta (4-8 Hz) and Beta (13-30 Hz)
    theta_idx = np.logical_and(freq >= 4, freq <= 8)
    beta_idx = np.logical_and(freq >= 13, freq <= 30)
    
    theta_power = np.trapz(psd[theta_idx], freq[theta_idx])
    beta_power = np.trapz(psd[beta_idx], freq[beta_idx])
    
    ratio = theta_power / beta_power
    return ratio

# Typical usage
tbr = theta_beta_ratio(cz_signal)
print(f"Theta/Beta Ratio: {tbr:.2f}")
# Values >3-4 sometimes considered elevated (but high variability)
```

## Stress Monitoring

### Physiological Markers

**Heart Rate Variability (HRV)**:
- **Primary stress indicator**: Reduced HRV = higher stress
- **Time-domain**: RMSSD, SDNN
- **Frequency-domain**: HF power (parasympathetic activity)
- **Interpretation**: Higher HRV = better stress resilience

**EEG Patterns**:
- **Reduced alpha**: During acute stress
- **Increased beta**: Arousal, cognitive load
- **Frontal alpha asymmetry**: Can shift during stress

**Skin Conductance (GSR/EDA)**:
- **Tonic level**: Baseline arousal
- **Phasic responses**: Acute stress reactions
- **Recovery time**: How quickly return to baseline

**Other Biosignals**:
- **Heart rate**: Increases with stress
- **Respiration rate**: Increases, becomes irregular
- **Muscle tension (EMG)**: Increases (especially trapezius, frontalis)

### HRV Biofeedback

**Rationale**: Increase parasympathetic activity, improve stress regulation.

**Protocol**:
1. Measure baseline HRV
2. Teach paced breathing (~6 breaths/minute)
3. Provide visual feedback of HRV
4. Practice increasing HRV through breathing
5. Generalize to daily life

```python
# Example: HRV calculation from ECG
def calculate_hrv_metrics(rr_intervals):
    """
    Calculate HRV metrics from R-R intervals (in milliseconds).
    
    Parameters:
    rr_intervals: Array of inter-beat intervals (ms)
    
    Returns:
    dict: HRV metrics (RMSSD, SDNN, pNN50)
    """
    rr = np.array(rr_intervals)
    
    # RMSSD: Root mean square of successive differences
    diff_rr = np.diff(rr)
    rmssd = np.sqrt(np.mean(diff_rr**2))
    
    # SDNN: Standard deviation of NN intervals
    sdnn = np.std(rr)
    
    # pNN50: Percentage of intervals differing by >50ms
    pnn50 = (np.sum(np.abs(diff_rr) > 50) / len(diff_rr)) * 100
    
    return {
        'RMSSD': rmssd,  # Higher = more parasympathetic
        'SDNN': sdnn,    # Overall variability
        'pNN50': pnn50   # Higher = healthier
    }

# Usage
hrv_metrics = calculate_hrv_metrics(rr_intervals)
print(f"RMSSD: {hrv_metrics['RMSSD']:.1f} ms")
# Higher RMSSD indicates better stress resilience
```

### Evidence Base

**What's Supported**:
- HRV strongly correlates with stress and health outcomes
- HRV biofeedback has good evidence for stress reduction
- Multimodal assessment (EEG + HRV + GSR) more informative than single measure

**Applications**:
- Workplace stress management
- Athletic performance optimization
- Chronic stress monitoring
- Burnout prevention

## Sleep Management

### Sleep Stages

**Wake**:
- Beta and gamma activity (13-100+ Hz)
- Muscle tone present
- Eye movements (when eyes closed: alpha)

**N1 (Light Sleep)**:
- Transition to sleep
- Theta activity increases
- Slow rolling eye movements
- 5-10% of sleep time

**N2 (Light Sleep)**:
- Sleep spindles (12-14 Hz bursts, ~0.5 sec)
- K-complexes (large slow waves)
- 45-55% of sleep time

**N3 (Deep Sleep / Slow-Wave Sleep)**:
- Delta waves (0.5-4 Hz) predominant
- Restorative sleep, memory consolidation
- Hard to wake
- 15-25% of sleep time

**REM (Rapid Eye Movement)**:
- Beta/theta activity (similar to wake)
- Muscle atonia (paralysis)
- Rapid eye movements
- Vivid dreams
- 20-25% of sleep time

### Sleep Architecture

**Normal Sleep Cycle**:
- 90-110 minute cycles
- N1 → N2 → N3 → N2 → REM
- 4-6 cycles per night
- More deep sleep early, more REM later

**Sleep Quality Indicators**:
- **Sleep efficiency**: Time asleep / time in bed (>85% good)
- **Sleep latency**: Time to fall asleep (<30 min normal)
- **WASO**: Wake after sleep onset (<30 min good)
- **Sleep stage distribution**: Balance of stages

### Neural Markers of Sleep Disorders

**Insomnia**:
- Increased beta during sleep (hyperarousal)
- Reduced slow-wave sleep
- Fragmented sleep architecture
- High-frequency EEG throughout sleep

**Sleep Apnea**:
- Arousals from deep sleep
- Fragmented sleep stages
- Reduced REM and deep sleep
- Accompanied by respiratory events (need breathing monitoring)

**Circadian Rhythm Disorders**:
- Shifted sleep-wake timing
- Irregular sleep patterns
- Can assess with actigraphy + melatonin timing

### Sleep Monitoring with Consumer Devices

**Capabilities**:
- Estimate sleep stages (not as accurate as lab PSG)
- Total sleep time
- Sleep efficiency
- Wake episodes

**Limitations**:
- Cannot detect sleep apnea reliably
- Stage classification ~70-80% accurate (vs. ~95% for human scoring)
- Movement artifacts problematic
- Not diagnostic tools

**Practical Use**:
- Track trends over time
- Identify sleep patterns
- Monitor intervention effects
- Complement clinical assessment

```python
# Example: Simple sleep stage classification features
def extract_sleep_features(eeg_epoch, fs=256):
    """
    Extract features for sleep stage classification.
    
    epoch: 30-second EEG epoch (standard sleep scoring window)
    """
    from scipy.signal import welch
    
    freq, psd = welch(eeg_epoch, fs=fs, nperseg=256)
    
    # Define bands
    delta = (0.5, 4)
    theta = (4, 8)
    alpha = (8, 13)
    beta = (13, 30)
    
    def band_power(band):
        idx = np.logical_and(freq >= band[0], freq <= band[1])
        return np.trapz(psd[idx], freq[idx])
    
    # Extract powers
    delta_power = band_power(delta)
    theta_power = band_power(theta)
    alpha_power = band_power(alpha)
    beta_power = band_power(beta)
    total_power = delta_power + theta_power + alpha_power + beta_power
    
    # Features
    features = {
        'delta_ratio': delta_power / total_power,
        'theta_ratio': theta_power / total_power,
        'alpha_ratio': alpha_power / total_power,
        'beta_ratio': beta_power / total_power,
        'delta_theta_ratio': delta_power / theta_power,
    }
    
    return features

# Simple heuristic classification (real systems use ML)
def classify_sleep_stage(features):
    """Heuristic sleep stage classifier (simplified)."""
    if features['beta_ratio'] > 0.3:
        return 'Wake'
    elif features['delta_ratio'] > 0.5:
        return 'N3'  # Deep sleep
    elif features['theta_ratio'] > 0.3:
        return 'N1/REM'  # Would need EOG to distinguish
    else:
        return 'N2'  # Light sleep

# Use in sleep monitoring
stage = classify_sleep_stage(extract_sleep_features(epoch))
```

### Sleep Interventions

**Sleep Hygiene Education**:
- Consistent schedule
- Avoid stimulants (caffeine, nicotine)
- Optimize environment (dark, cool, quiet)
- Limit screen time before bed

**Neurofeedback for Sleep**:
- **SMR training** (12-15 Hz): May improve sleep quality
- **Alpha-theta training**: Relaxation, sleep onset
- Evidence is limited but promising

**Light Therapy**:
- Adjust circadian rhythm
- Bright light in morning for delayed sleep phase
- Blue-blocking in evening

## General Implementation Principles

### Assessment Before Intervention

**Clinical Assessment**:
- Structured interview
- Standardized rating scales
- Rule out other conditions
- Set baseline for comparison

**Neurophysiological Baseline**:
- Record resting-state EEG
- Measure relevant metrics
- Establish individual norms
- Identify target parameters

**Functional Assessment**:
- How symptoms affect daily life
- Specific goals for intervention
- Barriers to participation
- Support system

### Intervention Design

**Personalization**:
- Individual neural patterns vary
- Tailor protocol to specific profile
- Adjust based on response
- Not one-size-fits-all

**Multimodal Approach**:
- Combine neurofeedback with behavioral therapy
- Address lifestyle factors (sleep, exercise, diet)
- Teach coping skills
- Family/social support

**Monitoring and Adjustment**:
- Track symptoms regularly (weekly)
- Monitor neural metrics each session
- Adjust protocol if no progress after 10-15 sessions
- Set milestones and review

### Evidence-Based Practice

**Current Evidence Requirements**:
- See `evidence-based-validation.md` for systematic evaluation
- Reference project Evidence Book for neurotech claims
- Large RCTs needed for clinical recommendation
- Most applications still investigational

**Regulatory Status**:
- Most neurofeedback not FDA-approved for treatment
- Used "off-label" or as wellness application
- Distinction between medical device and wellness device
- See `neurotech-market.md` for regulatory landscape

**Ethical Considerations**:
- Informed consent about evidence level
- Don't oversell benefits
- Not replacement for standard care
- Refer to clinician if worsening

## Further Reading

- See `neuroscience-fundamentals.md` for neural basis of mental health
- See `evidence-based-validation.md` for evaluating intervention claims
- See `bci-systems.md` for neurofeedback implementation
- See `tools-and-packages.md` for analysis code
- See project Evidence Book for systematic review of neurotech effectiveness
