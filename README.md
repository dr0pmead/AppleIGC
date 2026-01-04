<p align="center">
  <img src="media/logo.png" width="400" alt="AppleIGC Logo">
</p>

# AppleIGC

Intel i225/i226 2.5GbE kernel driver for macOS.

[![macOS Support](https://img.shields.io/badge/macOS-12.0--26.2-blue.svg?style=for-the-badge&logo=apple)](https://developer.apple.com/macos/)
[![Platform](https://img.shields.io/badge/Platform-x86__64-orange.svg?style=for-the-badge)](https://github.com/apple/darwin-xnu)
[![Release](https://img.shields.io/badge/Release-v1.8.0--Stable-green.svg?style=for-the-badge)](https://github.com/dr0pmead/AppleIGC)

This is a fork of the AppleIGC driver (v1.8.0) specifically optimized for stability on modern macOS releases, including the Sequoia and macOS 26 (Tahoe) developer previews. It addresses critical memory corruption issues and power state synchronization bugs present in earlier implementations.

---

## Stability Improvements

The primary goal of this fork is to eliminate random kernel panics and system freezes that occur under heavy I/O or power transitions.

### Kernel Panic Resolution (M_PKTHDR)
Older builds frequently triggered the `assertion failed: m->m_flags & M_PKTHDR` panic. This was traced to the driver passing "dirty" or malformed mbuf descriptors from the hardware RX ring directly to the kernel's network stack.
*   **Packet Duplication:** Implemented `mbuf_dup` in the receive path. By creating a clean copy of each packet, we isolate the kernel from any hardware-level descriptor corruption.
*   **Validation Layer:** Malformed packets (missing flags or zero length) are now discarded immediately, preventing corruption from reaching higher-level protocols.

### Power Management (setPowerState)
Proper support for Suspend/Resume has been implemented via the `setPowerState` method.
*   **Graceful Termination:** The driver now explicitly calls `igc_down` on sleep to stop the NIC and clear hardware registers.
*   **Clean Re-initialization:** On wake, `igc_up` is used to hardware-reset the controller and re-allocate ring buffers. This prevents desynchronization that previously led to hangs after sleep.

### Driver Optimizations
*   **EEE (Energy Efficient Ethernet):** Disabled to prevent link-state bouncing and latency spikes common on i225/i226 controllers.
*   **TSO (TCP Segmentation Offload):** Disabled to improve throughput stability and prevent buffer-related stalls.

---

## Verified Configuration

This driver has been extensively tested on the following setup:

| Component | Detail |
| :--- | :--- |
| **CPU** | Intel Core i7-14700KF |
| **GPU** | AMD Radeon RX 5700 XT |
| **OS** | **macOS 26.2 Tahoe** (Build 25C56) |
| **NIC** | Intel i225-V / i226-V |

---

## Installation

1. Download the latest `AppleIGC.kext` (v1.8.0) from the [Releases](https://github.com/dr0pmead/AppleIGC/releases) section.
2. Add the corresponding entry to your `config.plist`.
3. Add `e1000=0` to your boot-args to ensure the system uses this driver instead of the native one.

---

## Credits
This project is based on the work of:
- [songxiaoxi/AppleIGC](https://github.com/songxiaoxi/AppleIGC)
- [Shaneee/AppleIGB](https://github.com/Shaneee/AppleIGB)
- The Linux `igc` driver implementation.

Licensed under **GPLv2**. Use at your own risk.
