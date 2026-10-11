# Device requirements and confirmation

**Retroid Pocket 6 is the only tested device.**
Other native ARM64 ROCKNIX devices require an automatic untested-device
confirmation. Versioned and development builds use the same eligibility and
confirmation rules. Kernel eligibility is not proof of usable graphics,
audio, touch, controls or recovery on another handheld.

## Eligibility and installation

The installer requires native ROCKNIX, `aarch64` and a valid device-tree model.
The host-owned resolver discovers device nodes by their kernel capabilities.
Model and SoC names do not select or qualify graphics and video endpoints.

| Endpoint | Qualification |
| --- | --- |
| DRM render node | Exactly one eligible, direct, root-owned character node with matching sysfs device numbers and a successful DRM API query |
| V4L2 decoder | Optional, unique stateful memory-to-memory decoder with compressed H.264 OUTPUT and raw CAPTURE formats |

Missing or ambiguous render nodes block startup. Cameras, encoders and
stateless-only H.264 interfaces do not qualify as decoders. If no unique
decoder meets these kernel requirements, Desktop reports the reason, omits
the decoder mount and uses software decoding.

Discovered nodes retain their standard paths inside the container.
For example, `/dev/dri/renderD129` remains `/dev/dri/renderD129`, and
`/dev/video3` remains `/dev/video3`. Packaged Mesa selects its drivers
automatically. Kernel checks do not establish userspace acceleration or
compatibility with packaged consumers. Rendering and reliable MPV/Firefox
playback require hardware qualification.

[Optional native FEX reuse](fex.md) retains separate RP6 qualification.
Generic discovery does not enable it on another model.

Exit Desktop and leave EmulationStation running. Read the
[installation/data-replacement rules](../README.md#install-or-update) first.
Use the [public installer command](../README.md#install-or-update). RP6
proceeds normally. For every Install/Update invocation on an eligible non-RP6
model, the installer asks whether to proceed at your own risk before download
or installation changes. The answer defaults to **No**; `--yes` cannot bypass
this warning. Unless `--yes` was supplied, a separate Install/Update confirmation
describes data replacement or preservation.

`--check` checks installer prerequisites only: it does not download the bundle,
check GPU/decoder endpoints or battery, or assess installation health. It does
not prompt and reports when installation will require device confirmation. Normal
Install/Update verifies the downloaded bundle's device check before classifying
or replacing the installation. A bundle without untested-device support
is refused before Desktop data replacement. Resolver failures retain
`device-check.log` in the reported staging path; no Desktop data is replaced
by a device-check refusal.
Other preflight/runtime requirements (compositor, input, services, namespaces,
mounts and resources) still apply after confirmation.

## Input and power requirements

Desktop selects one InputPlumber controller composite by its capabilities
and source identity. Configuration, profile switching and recovery use that
same recorded identity. An ambiguous or replaced controller blocks mutation.
Desktop does not assume device zero. Physical touch keeps the host Sway
mapping, without a Desktop touchscreen-to-output override.

Install and Update require exactly one positively identified system battery
with at least 10% charge, even while charging. Peripheral batteries do not
count. Missing, ambiguous, invalid or unknown telemetry blocks maintenance.
The standalone installer checks before download. Installed maintenance helpers
check again before writes. A device without battery telemetry does not qualify
solely because AC power is connected.

## Consent and lifecycle

Installation stores model-bound consent at
`managed/host/state/device.json`, outside guest mounts. The record and ancestors
must be root-controlled; the record is mode 0600 and grants no configurable
mount paths, commands or guest-supplied profile selection.

Installed consent lets the launcher/preflight and internal maintenance tools
recognize the same model. Every public Install/Update invocation on that
non-RP6 model still asks for the device confirmation.
Uninstall checks identity/consent without requiring live GPU/decoder endpoints,
so removal remains possible when those endpoints are unavailable. Uninstall
retains data/tools/consent; replacement Install removes the managed/data trees
and records consent again after device confirmation.

## Reporting a problem

Include the device model, ROCKNIX build, installed Desktop version from
**About Desktop Mode**, and the failed operation or error message. Remove
private network details from logs and screenshots before sharing them.

Widening access to all devices, storage or sockets is not a compatibility
workaround.
