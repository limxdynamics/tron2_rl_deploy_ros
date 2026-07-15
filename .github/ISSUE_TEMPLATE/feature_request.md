---
name: Feature request
about: Suggest an improvement to the RL deployment stack
title: "[feat] <short summary>"
labels: enhancement
assignees: ''
---

## Problem

<!-- What is missing or awkward in the current deployment stack? -->

## Proposed change

<!-- Files, launch files, controllers, or interfaces you propose adding
     or editing. -->

## Alternatives considered

## Downstream impact

- Packages affected: `tron2_hw` / `tron2_controllers` / `robot_common` /
  `onnxruntime_sdk` / meta
- Simulation vs real hardware: which paths does this touch?
- Backwards compatibility (topic / service / parameter renames?):
- Sibling repositories touched (`robot-description`, `limxsdk-lowlevel`)?

## Checklist

- [ ] This request does **not** require shipping vendor CAD, firmware,
      calibration values, new control policies (`*.onnx`), new SDK
      binaries, or motion data (`*.bag`, `*.mcap`) without prior
      owner approval.
- [ ] This request does **not** introduce auto-start behaviour on real
      hardware.
- [ ] I have skimmed `CONTRIBUTING.md` for directory / naming
      conventions and the model / binary provenance policy.
