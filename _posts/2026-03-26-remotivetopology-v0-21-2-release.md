---
layout: default
title: "RemotiveTopology v0.21.2: Selective Signal Decoding and Instance Generation"
date: 2026-03-26
categories: [RemotiveTopology, Release]
---

RemotiveTopology v0.21.2 is now available, bringing two powerful features that streamline your workflow: selective signal decoding and the ability to save generated instance files.

## Selective Signal Decoding

When working with large CAN frames containing dozens of signals, you often only need a few specific signals for your test or analysis. The new `--only` option lets you decode just the signals you care about.

**Before:**
```bash
# Decode entire frame with all 20+ signals
remotive topology decode frame 0x123
```

**Now:**
```bash
# Decode only the signals you need
remotive topology decode frame 0x123 --only VehicleSpeed,EngineRPM
```

This makes signal inspection faster and output cleaner, especially when debugging specific issues or monitoring key signals during testing.

![Selective signal decoding example](../assets/images/selective-decode-example.png)
*Decode only the signals you need from complex frames*

## Save Generated Instance Files

Previously, `remotive topology generate` would create your instance configuration in memory for immediate use. Now you can save it for reuse and version control:

```bash
# Generate and save the instance file
remotive topology generate -f platform.yaml --save-instance lighting.instance.yaml

# Later, reuse the same configuration
docker compose -f build/lighting/docker-compose.yaml up
```

This is valuable for:
- **Version control** - Track instance configurations alongside platform definitions
- **Collaboration** - Share exact test setups with your team
- **CI/CD pipelines** - Reuse validated configurations across test runs
- **Documentation** - Keep a record of working test configurations

## FlexRay Improvements

This release also includes significant FlexRay enhancements:
- Optimized virtual FlexRay frame publishing for better performance
- Proper signal start value initialization
- More reliable FlexRay channel simulation

## Bug Fixes

- Fixed relative path generation in instance files
- Improved Docker health check reliability
- Clearer error messages when including incompatible file types

## Upgrade

Update to the latest version:

```bash
pipx upgrade remotivelabs-cli
```

Check your version:

```bash
remotive topology --version
```

---

*See the [full changelog](https://github.com/remotivelabs/remotivelabs-cli/releases/tag/remotive-topology-v0.21.2) on GitHub or read the [RemotiveTopology documentation](https://docs.remotivelabs.com/docs/remotive-topology) to learn more.*
