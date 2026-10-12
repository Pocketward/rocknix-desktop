# Changelog

## v0.2.6

- Make installation and updates independent of the caller's umask. A fresh installation also restores access through a project directory left by an earlier restrictive installation.
- Preserve the original InputPlumber target order when restoring native controller input.

## Highlights since v0.2.0

- Discover graphics, optional H.264 decoding, controller identity and system battery on native ARM64 ROCKNIX. Allow software video fallback when no decoder qualifies and preserve untested-device consent. RP6 remains the only tested device.
- Keep RP6 panel sizing confined to its internal display and use generic sizing for other outputs.
- Isolate Desktop-launched Steam shared memory so stopped sessions cannot accumulate new host allocations, and add Show Steam for running Keep Desktop sessions.
- Fix Gamescope SDL shutdown, coordinate window visibility with Vulkan presentation, and bound child-process cleanup when an application closes or the compositor crashes.
- Make Apps and Settings menus dismiss reliably, preserve manual controller overrides, and correct Fuzzel selection when clicking immediately after pointer entry.
