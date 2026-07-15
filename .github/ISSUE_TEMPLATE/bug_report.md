---
name: Bug report
about: Defect in the RL controller, hardware interface, or launch / bring-up flow
title: "[bug] <short summary>"
labels: bug
assignees: ''
---

## Affected package / file(s)

<!-- e.g. tron2_controllers/src/foo.cpp, tron2_hw/launch/tron2_hw.launch -->

- Package(s): `tron2_hw` / `tron2_controllers` / `robot_common` / `onnxruntime_sdk` (pick)
- File(s):
- Robot variant: `SF_TRON2A` / `WF_TRON2A` / other
- Commit / tag:

## Deployment mode

- [ ] Gazebo simulation (`tron2_hw_sim.launch`, `tron2_controller_sim.launch`)
- [ ] Real hardware (`tron2_hw.launch`) — **hoisted / on-stand**
- [ ] Real hardware — free-standing (safety review required before reproducing)

## Environment

- ROS distribution: Noetic / other
- OS + arch: Ubuntu 20.04 x86_64 / arm64 / …
- ONNX Runtime version (from `onnxruntime_sdk/lib/`):
- Sibling repos: `robot-description` commit …, `limxsdk-lowlevel` commit …

## Expected behavior

<!-- What the controller / hardware interface should do. -->

## Actual behavior

<!-- Log lines, screenshots, numeric mismatches, service replies. -->

## Minimal reproduction

```bash
# commands that reproduce it, starting from a freshly built workspace
```

## Additional context

<!-- Cross-links to related issues, upstream ROS / ONNX Runtime quirks. -->

## Checklist

- [ ] I have searched existing issues.
- [ ] I have included the exact commit / tag.
- [ ] I am **not** reporting a security or robot-safety issue.
      Those go to `contact@limxdynamics.com` per `SECURITY.md`.
