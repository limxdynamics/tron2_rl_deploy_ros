# Security Policy

## Scope

`tron2-rl-deploy-ros` is the **ROS-side deployment stack** for the
TRON2A biped. Unlike the sibling `robot-description` repository, this
one **does** contain:

- Real-hardware control code (`tron2_hw`), including a
  controller-manager loop and start / stop / emergency-stop services.
- An RL controller (`tron2_controllers`) that loads ONNX policy and
  encoder weights, applies torques, and drives joints on a physical
  humanoid.
- A bundled ONNX Runtime shared library (`onnxruntime_sdk`).

The security surface is therefore larger than a pure asset repo, and
mistakes here can move a real 1-plus-metre-tall bipedal robot in a
lab. Please treat reports accordingly.

Primary concerns:

- **Robot-safety-relevant defects.** Anything that could cause the
  emergency-stop service to fail, the controller loop to stall, the
  policy output to be applied while an unsafe state is active, or a
  bad ONNX file to be silently loaded and executed.
- **Supply-chain integrity of the bundled binary.** Tampering with
  `onnxruntime_sdk/lib/libonnxruntime.so` between publication and
  consumption would let attackers execute code inside the control
  process.
- **Model / weight tampering.** Substituting a different `policy.onnx`
  or `encoder.onnx` at deploy time to induce unsafe motion.
- **Disclosure of internal infrastructure metadata** (private IPs,
  hostnames, credentials, or on-robot secrets) via source, launch
  files, docs, or media.

Purely descriptive vulnerabilities (URDF / mesh / calibration
disclosure) belong to `robot-description`.

## Supported versions

Only the tip of the `main` branch and the most recent tagged release
receive security fixes. Older tags are provided as-is.

| Version    | Supported |
|------------|-----------|
| `main`     | ✅        |
| Latest tag | ✅        |
| Older tags | ❌        |

## Reporting a vulnerability

**Do not** open a public issue for security reports.

Email: **contact@limxdynamics.com**
Subject prefix: `[tron2-rl-deploy-ros]`

Please include:

- Affected file(s) and commit / tag.
- A minimal reproducer or proof of concept.
- Impact assessment. If the issue can move real hardware or defeat
  the emergency-stop service, say so explicitly in the first line so
  we can route it to the robot-safety owner.
- Your preferred disclosure timeline and contact.

We aim to acknowledge reports within **3 business days** and provide a
remediation plan or an initial mitigation within **14 calendar days**.
For issues that require physical mitigations (bring-up procedure
changes, service ACLs, hardware interlocks), we may extend the
disclosure window in coordination with the reporter.

## Real-hardware safety notes (informational, not a substitute for
lab procedure)

These are project-level defaults. Site-specific safety procedures
override them.

- Always bring the robot up **hoisted / suspended** for the first run
  of any new policy or controller build. This is called out in
  `README.md` and reflected in `doc/deploy.jpg`.
- The controller-manager exposes start, stop, and emergency-stop
  services (see `tron2_hw/docs/bringup_mvp.md`). The emergency-stop
  service is intended to be the fastest way to halt motion from
  software; it is **not** a substitute for a hardware e-stop.
- Do **not** load a `policy.onnx` / `encoder.onnx` pair whose SHA-256
  is unknown or not attested by the model owner. Report substitution
  attempts as a security issue.
- Do **not** replace `onnxruntime_sdk/lib/libonnxruntime.so` with a
  binary whose SHA-256 you have not verified against the value
  recorded in `THIRD_PARTY_NOTICES.md`.
- Report any launch file, service, or driver that starts controllers
  automatically on boot without an explicit operator action — auto-
  start on a physical humanoid is a robot-safety defect, not a UX
  choice.

## Out of scope

- Bugs in third-party components (ROS, ros-control, ONNX Runtime,
  Gazebo) — report those upstream.
- Physical safety of the robot mechanism itself — report to LimX
  product support at `contact@limxdynamics.com`.
- Requests to publish training data, calibration data, or firmware —
  this repository intentionally excludes those.

## Safe harbor

Good-faith security research that follows this policy will not be
pursued legally by LimX Dynamics. Please respect user privacy, avoid
service disruption, and — most importantly — do not test control-path
vulnerabilities on a robot that is not properly hoisted and inside a
supervised lab. Simulated reproducers (Gazebo) are strongly preferred.
