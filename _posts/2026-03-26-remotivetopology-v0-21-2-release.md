---
layout: default
title: "RemotiveTopology v0.21.2: Selective Signal Decoding"
date: 2026-03-26
categories: [RemotiveTopology, Release]
parent: RemotiveTopology
---

RemotiveTopology v0.21.2 is now available, bringing two powerful features that streamline your workflow: selective signal decoding and the ability to save generated instance files.

## Full change log

**Features**

- decoder: Add only option to selectively decode signals from a frame (a5689ef)
- Save generated instance file (6d6e3d5)

**Bug Fixes**

- decoder: Use SOME/IP frame with meta fields in selective decode test (f40f593)
- Clarified e005 when including wrong file type (5f4233b)
- Set content type for docker health checks (6a7802a)
- Optimized how virtual flexray frames are published (9a1cfa4)
- Flexray initialize signal start values (81c974d)
- Generated instance file with correct relative paths (b918be3)

**Miscellaneous Tasks**

- topology: Bump default broker version (b21a7cf)
- Corrected signal ordering (0494c88)
- Use Encoder.field_value() to represent field with value (6adf418)

---

*See the [full changelog](https://github.com/remotivelabs/remotivelabs-cli/releases/tag/remotive-topology-v0.21.2) on GitHub or read the [RemotiveTopology documentation](https://docs.remotivelabs.com/docs/remotive-topology) to learn more.*
