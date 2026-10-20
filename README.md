# SeedSigner OS — XBT fork build (unofficial)

**Unofficial. Not affiliated with, reviewed by or endorsed by the SeedSigner project.** This branch (`xbt`) is upstream SeedSigner OS `v0.8.7`, unchanged except for this note. The fork is in the app the OS boots: [Kilombino/seedsigner](https://github.com/Kilombino/seedsigner), branch `xbt`. That fork adds **XBT mode**, which signs with sighash `0x21` for the Bitcoin BLAKE2b chain, **JoinMarket fidelity-bond spending** and **input-amount verification**. Its README explains them.

**Untested on real hardware by us.** Use a seed with no real funds first.

## Rebuild a release image and compare

Same procedure as upstream's [reproducible build](docs/building.md), pointed at the fork's tag:

```
git clone https://github.com/Kilombino/seedsigner-os.git && cd seedsigner-os
git checkout xbt && git submodule update --init --depth 1
export BOARD_TYPE=pi0        # or pi02w, pi2, pi4
SS_ARGS="--$BOARD_TYPE --app-repo=https://github.com/Kilombino/seedsigner.git --app-branch=v0.8.7-xbt1" docker compose up --force-recreate --build
sha256sum images/seedsigner_os.v0.8.7-xbt1.$BOARD_TYPE.img
```

Compare the hash with `SHA256SUMS` in the [release](https://github.com/Kilombino/seedsigner/releases). Then check that file's signature with the fork's key, [`seedsigner-xbt_pubkey.asc`](https://github.com/Kilombino/seedsigner/blob/xbt/seedsigner-xbt_pubkey.asc), fingerprint `DF04 5303 59B6 BDC8 2E05  66D7 3C2B C6F7 4555 C3D1`:

```
gpg --import seedsigner-xbt_pubkey.asc
gpg --verify SHA256SUMS.asc SHA256SUMS
```

The same procedure, with `--app-branch=0.8.7` and no `--app-repo`, reproduces upstream's own 0.8.7 images. That is how we checked that this environment builds reproducibly.

---

<p align="center">
  <a href="https://seedsigner.com/">
    <img alt="Gitea" src="docs/img/logo.png" width="90"/>
  </a>
</p>
<h1 align="center">SeedSigner OS</h1>

<p align="center">
  <a href="https://opensource.org/licenses/MIT" title="License: MIT">
    <img src="https://img.shields.io/badge/License-MIT-blue.svg">
  </a>
  <a href="" title="Twitter">
  <img src="https://img.shields.io/twitter/follow/seedsigner?style=social">
  </a>
</p>

* [Overview](#overview)
* [Building](docs/building.md)
* [Building (without Docker)](docs/without_docker.md)
* [SeedSigner OS structure](docs/structure.md)
* [Dev workflow](docs/dev_workflow.md)
* [Customizing Buildroot](docs/customize_buildroot.md)

<br/>

JUMP STRAIGHT TO: [🔥🔥🔥🛠 Quickstart: SeedSigner Reproducible Build! 🛠🔥🔥🔥](docs/building.md)

<br/>

---

# Overview

A custom linux based operating system built to manage software running on airgapped Bitcoin signing device. SeedSigner is both the project name and [application](http://github.com/SeedSigner/seedsigner/) running on airgapped hardware. This custom operating system, like all operating systems, manages the hardware resources and provides them to the application code. It's currently designed to run on common Raspberry Pi hardware with [accessories](https://github.com/SeedSigner/seedsigner/#shopping-list). The goal of SeedSigner OS is to provide an easy, fast, and secure way to build microSD card image to securely run [SeedSigner](https://seedsigner.com) code.


## ⚙️ Under the Hood

SeedSigner OS is built using [Buildroot](https://www.buildroot.org). Buildroot is a simple, efficient and easy-to-use tool to generate embedded Linux systems through cross-compilation. SeedSigner OS does not fork Buildroot, but uses Buildroot with custom configurations to build microSD card images tailor made for running SeedSigner.


## 🛂 Security

SeedSigner OS is built to reduce the attack surface area and enable additional application functionality. The OS is an order of magnitude smaller in size than Raspberry Pi OS (which is what typically is used to run software on a Pi device). Here are a list of some security and functional advantages of using SeedSigner OS.

- Boots 100% from RAM. This means, once you see the SeedSigner splash screen, you can remove the microSD card because no disk I/O is needed after boot!
- One FAT32 partition on the microSD card
- Removes these standard Raspberry Pi OS Kernel modules:
   - Networking and Bluetooth
   - SWAP
   - I2C
   - Serial
   - USB
   - Pulse-Width Modulation (PWM)
- NO HDMI support
- NO Serial connection TTL support
- NO Software supporting any wireless or networking chips
- A single read only zImage file on the boot partition containing the entire Linux kernel and filesystem


