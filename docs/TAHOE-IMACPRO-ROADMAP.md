# 「 BIG MOUTH STRIKES AGAIN 」— macOS Tahoe Roadmap

## Target: iMac Pro 2017 (iMacPro1,1) → macOS Tahoe
## Secondary Use: File Server + Retina Relay Display (retinarelay.com)

> *"Sweetness, sweetness I was only joking when I said I'd like to smash every Mac into Tahoe"*

---

## Upstream OCLP Forks to Track

| Fork | Author | Focus | Status |
|------|--------|-------|--------|
| [OCLP-X](https://github.com/JeoJay127/OCLP-X) | JeoJay127 | Extended Tahoe support | Active |
| [OCLP-Mod](https://github.com/laobamac/OCLP-Mod) | laobamac | Modified OCLP builds | Active |
| [OCLP-Plus](https://github.com/YBronst/OCLP-Plus) | YBronst | Tahoe patchset (our upstream) | Active |

**Action: Cherry-pick Tahoe-specific fixes from all three forks weekly.**

---

## 🔴 CRITICAL ISSUES (Blocking Boot)

### Issue #1: T2 Controller Panic & "Missing" NVMe

**Priority:** Critical / Blocker
**Status:** ❌ Failing at Kernel Init

**The Problem:**
The iMac Pro 2017's internal NVMe SSD is physically routed through the T2 Security Chip. macOS Tahoe's kernel has purged the specific handshake protocol required to "talk" to first-gen T2 controllers.

**Symptoms:**
- Installer boots but sees ZERO internal disks
- If forced to mount: kernel panic (Unknown Controller response)
- Infinite boot loop

**Required Fix:**
Custom Lilu plugin or ACPI patch (SSDT) that intercepts the hardware probe and spoofs the T2 controller as a generic NVMe bridge or a supported 2020 Intel controller.

**Deliverables:**
- [ ] `SSDT-T2-SPOOF.dsl` — ACPI spoof for T2 NVMe bridge
- [ ] Test with `NVMeFix.kext` v1.1.2 + custom SSDT
- [ ] Check OCLP-X and OCLP-Mod for any T2 patches
- [ ] If SSDT fails: investigate Lilu plugin approach

---

### Issue #2: Radeon Pro Vega Metal Acceleration

**Priority:** Critical
**Status:** ❌ Software Rendering Only

**The Problem:**
Apple removed the legacy Metal binaries for Radeon Pro Vega 56/64. Tahoe forces a new graphical memory structure that the GPU drivers don't support.

**Symptoms:**
- OS boots with sluggish "glass" UI (~3-5 FPS)
- Animations run in software rendering via CPU
- Windows tear, Dock glitchy, Maps/Photos crash

**Required Fix:**
Extract Metal graphics binaries from the macOS Sequoia KDK, recompile to bypass Tahoe's library validation, inject via custom WhateverGreen build.

**Deliverables:**
- [ ] Extract Vega Metal binaries from Sequoia 15.5 KDK
- [ ] Patch library validation signatures
- [ ] Build custom `WhateverGreen.kext` with Vega support
- [ ] Test with `AMDRadeonX5000.kext` injection
- [ ] Check OCLP-Plus `amd_vega.py` patchset for updates

---

## 🟠 HIGH PRIORITY ISSUES

### Issue #3: "Skywalk" Network Stack

**Priority:** High
**Status:** ❌ Hardware Unrecognized

**The Problem:**
Apple replaced the legacy Wi-Fi stack (IO80211Family) with "Skywalk" (IOSkywalkFamily). The iMac Pro's Broadcom Wi-Fi card is incompatible.

**Symptoms:**
- Wi-Fi icon grayed out or missing
- Bluetooth unavailable

**Required Fix:**
1. Block `IOSkywalkFamily.kext` in boot config
2. Inject legacy Wi-Fi drivers from macOS Sonoma/Sequoia
3. Patch AMFI to allow unsigned older network drivers

**Deliverables:**
- [ ] Add `IOSkywalkFamily` block to config.plist
- [ ] Extract `IO80211FamilyLegacy` from Sonoma
- [ ] Patch AMFI via `AMFIPass.kext`
- [ ] Test Bluetooth with `BlueToolFixup.kext`
- [ ] Verify with `IO80211ElCap-v2.0.1` in our payloads

---

### Issue #4: T2 Audio Routing Dead-End

**Priority:** Medium
**Status:** ❌ No Output Devices

**The Problem:**
On iMac Pro, the T2 chip handles audio processing (DAC). Tahoe lacks the `AppleHDA` controller configuration to send audio to the T2 chip.

**Symptoms:**
- Volume icon grayed out
- "No Output Devices Found" in System Settings

**Required Fix:**
Rewritten `AppleALC` layout that manually maps the T2 audio bridge, forcing the OS to send raw audio data to the T2 chip.

**Deliverables:**
- [ ] Create custom `AppleALC` layout for T2 audio bridge
- [ ] Map audio codec paths: OS → T2 DAC → speakers
- [ ] Test with `AppleALC-v1.6.3` + custom layout-id
- [ ] Check if USB audio works as fallback

---

## 📋 PROJECT ROADMAP

### Phase 0: Setup (This Week)
- [ ] Install Retina Relay on iMac Pro (use as display from M4 Max)
- [ ] Set up file server (SMB/NFS shares)
- [ ] Verify current macOS version on iMac Pro
- [ ] Back up current system
- [ ] Fork latest OCLP-X, OCLP-Mod, OCLP-Plus for reference

### Phase 1: Storage (Next Week — Blocker)
- [ ] Generate `SSDT-T2-SPOOF.dsl`
- [ ] Test NVMe visibility with spoofed T2
- [ ] If successful: install macOS Tahoe to internal SSD
- [ ] If blocked: test with external USB/Thunderbolt SSD boot

### Phase 2: Graphics (Week 2)
- [ ] Extract and patch Vega Metal binaries
- [ ] Build custom WhateverGreen
- [ ] Test Metal acceleration
- [ ] Benchmark: verify hardware-accelerated rendering

### Phase 3: Network (Week 2-3)
- [ ] Block Skywalk, inject legacy Wi-Fi
- [ ] Patch AMFI for unsigned drivers
- [ ] Test Wi-Fi and Bluetooth connectivity

### Phase 4: Audio (Week 3)
- [ ] Custom AppleALC layout for T2 audio
- [ ] Test internal speakers + headphone jack
- [ ] USB audio fallback if needed

### Phase 5: Polish & Community (Week 4)
- [ ] Automated build script for iMac Pro Tahoe
- [ ] Document all patches and workarounds
- [ ] Create installer package
- [ ] Community release + guide

---

## iMac Pro 2017 Specs (iMacPro1,1)

| Component | Spec |
|-----------|------|
| CPU | Intel Xeon W-2140B (8-core, 3.2 GHz) |
| GPU | Radeon Pro Vega 56 (8GB HBM2) |
| RAM | 32GB DDR4 ECC (expandable to 256GB) |
| Storage | 1TB NVMe SSD (T2-routed) |
| T2 Chip | First-generation Apple T2 Security Chip |
| Display | 27" 5K Retina (5120×2880) |
| Wi-Fi | Broadcom BCM94360CD (802.11ac) |
| Thunderbolt | 4× Thunderbolt 3 (USB-C) |
| Ethernet | 10Gb Nbase-T |

## Dual Use Plan

```
PRIMARY:   File server (SMB/NFS) for Star Platinum cluster
SECONDARY: Retina Relay display for M4 Max (retinarelay.com)
TERTIARY:  macOS Tahoe testing / community contribution
```

---

## Related Resources

- [Retina Relay](https://www.retinarelay.com/) — Use iMac as external display
- [OCLP Documentation](https://dortania.github.io/OpenCore-Legacy-Patcher/)
- [iMac Pro Service Manual](https://support.apple.com/en-us/111897)
- [T2 Security Chip Overview](https://support.apple.com/en-us/HT208862)

---

*Stand User: RavenX LLC / @DeadByDawn101*
*"We don't give up. We do what others don't and build what isn't possible."*
