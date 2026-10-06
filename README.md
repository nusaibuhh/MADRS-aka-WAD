# Multimodal Anxiety Detection & Relief System

A system that combines **digital behavior data** (mouse cursor trajectory, typing patterns, blink rate, and similar signals) with **physiological data** (PPG, EEG) to detect signs of anxiety in real time, and responds with a lightweight **intervention** such as a guided breathing exercise or something else (not fixed yet).

The long-term goal is a **wearable / hearable device paired with software** that works passively in the background, learns each person's own baseline, and offers help when their signals shift in ways associated with anxiety.

> **Status: early research prototype.** No public datasets are currently available for this problem, so the first prototype is **rule-based** and **personal-baseline driven**. It is not a medical device and does not diagnose any condition.

---

## Vision

| Layer | What it does |
|---|---|
| **Sensing** | Collects digital behavioral signals from the computer and physiological signals from a wearable/hearable |
| **Personal baseline** | Learns what "normal" looks like for each user, since typing, cursor, and physiological patterns differ widely between people |
| **Detection** | Compares live signals to the baseline and flags sustained shifts in the anxiety direction |
| **Intervention** | Offers relief when anxiety indicators are elevated, e.g. a paced breathing exercise |
| **Feedback loop** | Records whether the user found the intervention helpful, to tune thresholds and later train models |

## Signals

### Digital behavior (software only)

| Source | Examples of features |
|---|---|
| Keyboard | Key-hold (dwell) time, inter-key (flight) time, typing speed, backspace/error rate, pause ratio (stop-and-go typing), digraph timing |
| Mouse | Speed and acceleration profile, path directness, click accuracy, hover duration, scroll reversals, idle time, screen coverage |
| Eyes (planned, webcam) | Blink rate and related blink metrics |

### Physiological (wearable / hearable hardware)

| Sensor | Typical derived measures |
|---|---|
| PPG | Heart rate, heart rate variability (HRV) |
| GSR / EDA | Skin conductance level and responses |
| EEG | Band power and related spectral features |

Other sensors (skin temperature, respiration, motion for artifact rejection) may be added later.

## Roadmap

| Phase | Scope | Status |
|---|---|---|
| **1. Keyboard & mouse dynamics** | Rule-based detector with calibration and monitoring (this repo's current prototype) | In progress |
| **2. Additional digital signals** | Webcam blink rate and other behavioral signals | Planned |
| **3. Physiological sensing** | PPG, GSR, EEG acquisition on wearable/hearable hardware | Planned |
| **4. Multimodal fusion** | Combine digital and physiological features into one anxiety indicator | Planned |
| **5. Intervention module** | Breathing exercise and similar relief mechanisms, triggered by the detector | Planned |
| **6. Data collection & learning** | Collect labeled data with consent, then train and validate ML models against the rule-based baseline | Planned |

## Phase 1: Rule-Based Keyboard & Mouse Prototype

The first prototype is a local desktop tool (Python) that works in two stages:

1. **Calibration.** The user answers a few neutral prompted questions and clicks on-screen targets. The tool records keyboard and mouse timing and builds a personal baseline (mean and spread of each feature).
2. **Monitoring.** In fixed windows (default 20 s), the tool computes the same features and compares each to the baseline. A feature counts as "flagged" when it moves more than 1.5 standard deviations in its expected anxiety direction. A window is "elevated" when enough features are flagged, and the alert fires only when elevation is **sustained** (3 of the last 5 windows).

Expected direction of change under anxiety (from the literature, effects are small and vary between people):

| Feature | Expected change |
|---|---|
| Dwell time | Increases |
| Flight-time variability | Increases |
| Typing speed | Decreases |
| Backspace / error rate | Increases |
| Pause ratio | Increases (more stop-and-go) |
| Mouse speed variability | Increases |
| Path directness | Decreases |
| Idle time | Increases |
| Scroll direction reversals | Increases |
| Screen coverage | Increases |

**Files**

- `anxiety_monitor.py`: command-line version (`calibrate` / `monitor`)
- `anxiety_monitor.ipynb`: the same prototype as a notebook

### Quick start

```bash
pip install pynput numpy

python anxiety_monitor.py calibrate   # about 3 minutes, writes baseline.json
python anxiety_monitor.py monitor     # runs until Ctrl+C
```

Run it on a local desktop session. On Linux use X11 (pynput does not capture globally on Wayland). On macOS, grant Input Monitoring and Accessibility permission to your terminal or Jupyter app.

## Why rule-based first?

- **No public dataset** exists for this multimodal setup, so supervised ML is not yet possible.
- Behavioral and physiological signals are **highly individual**, so a per-user baseline is needed regardless of the final model.
- A rule-based system is transparent and easy to debug, and it produces the **logged, labeled data** needed to train and validate learned models later.

## Known limitations

- **Context confounds.** Typing and mouse patterns change with the task (coding vs. writing vs. browsing), fatigue, device, and skill, not only with anxiety.
- **Not yet validated.** Without ground-truth labels, a flag means "behavior shifted from baseline in a stress-associated direction", not "the user is anxious".
- **Small, individual effects.** Findings in the literature vary a lot between people, so thresholds need per-user tuning.

## Validation plan

- Collect sessions together with self-report measures (e.g. GAD-7, STAI) from consenting participants.
- Include a mild, ethically approved stress task (e.g. timed mental arithmetic) and a relaxed control task, and check that flags rise in the former and not the latter.
- Evaluate with **subject-wise** cross-validation so models are tested on people they have not seen.

## Privacy & ethics

- **Local-first.** Processing and storage stay on the user's device.
- **Timing only.** The keyboard module records timestamps and whether a key was Backspace, never which characters were typed.
- **Informed consent** is required for any data collection, and participants should be able to review and delete their data.
- **Wellbeing framing.** Outputs are presented as an indicator of stress-related behavioral change, not a diagnosis. Interventions are supportive tools, not treatment.

## Disclaimer

This project is a research prototype and is **not a medical device**. It must not be used to diagnose, treat, or monitor any medical or psychiatric condition. If you are struggling with anxiety or your mental health, please reach out to a qualified healthcare professional.
