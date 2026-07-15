<!--
Thanks for contributing to tron2-rl-deploy-ros!
Please fill in the sections below. Delete any that are not applicable.
This repository contains code that can move a real humanoid — the
checklists exist for good reasons.
-->

## Summary

<!-- One paragraph: what and why. -->

## Type of change

- [ ] `fix`      — corrects a defect in the controller / hardware interface / launch flow
- [ ] `feat`     — new capability (controller, launch mode, parameter surface)
- [ ] `refactor` — non-behavioural code change
- [ ] `docs`     — README, THIRD_PARTY_NOTICES, SECURITY, CHANGELOG, or CONTRIBUTING
- [ ] `ci`       — GitHub Actions or verification tooling
- [ ] `safety`   — robot-safety fix (bring-up, emergency-stop, service ACL)
- [ ] `chore`    — repo maintenance (deps, formatting, cleanup)

## Affected packages

- [ ] `tron2_hw`
- [ ] `tron2_controllers`
- [ ] `robot_common`
- [ ] `onnxruntime_sdk`
- [ ] Meta / repo-wide

## Deployment path exercised

- [ ] Gazebo simulation only
- [ ] Real hardware, **hoisted** — attach video / bag reference in
      the PR body
- [ ] Real hardware, free-standing — **requires robot-safety owner
      review and lab sign-off**

## Verification

Paste the output (or a summary) of the local verification steps from
`CONTRIBUTING.md#verification-before-opening-a-pr`:

```text
catkin_make:                ...
xmllint on every package.xml: ...
license annotation scan:     ...
controlled-artifact scan:    ...
private-IP scan:             ...
```

## Provenance & licensing

<!-- Required if the PR touches any *.onnx, any *.so / *.dll / *.dylib
     / *.lib, any doc media, or any package.xml <license> line. -->

- [ ] **This PR does not add ONNX weights or new SDK binaries without
      a NOTICE update.** (Required.)
- [ ] If any `*.onnx` was added / changed: model owner sign-off is
      recorded, `THIRD_PARTY_NOTICES.md` §3 is updated, model card and
      SHA-256 are included.
- [ ] If any `*.so` / `*.dll` / `*.dylib` / `*.lib` was added /
      changed: SDK owner sign-off is recorded, upstream tag / build
      flags / SHA-256 are in `THIRD_PARTY_NOTICES.md` §2, and the
      upstream `LICENSE` file is vendored next to the binary.
- [ ] `<license>TODO` / `<license>Proprietary` lines are still
      preceded by the `⚠ TO CONFIRM` annotation comment.
- [ ] `doc/*` media (if changed) has been EXIF-stripped and reviewed
      for individuals, office locations, badges, and internal
      hostnames.

## Excluded artifacts

- [ ] This PR does **not** add: firmware, factory calibration values,
      rosbags / MCAP, HDF5 trajectory captures, customer data, or
      new private IPs / hostnames beyond the documented placeholder.

## Safety review (real hardware only)

- [ ] Robot-safety owner has reviewed changes under `tron2_hw/launch/`,
      `tron2_hw/src/`, and `tron2_hw/docs/bringup_mvp.md`.
- [ ] Emergency-stop service still halts motion within the documented
      time budget.
- [ ] No new auto-start behaviour introduced.

## Checklist

- [ ] `catkin_make` succeeds locally.
- [ ] `CHANGELOG.md` has an entry under `## [Unreleased]`.
- [ ] All commits are DCO-signed (`git commit -s`).
- [ ] CI is expected to pass.

## Related issues

<!-- Fixes #123 / Refs #456 -->
