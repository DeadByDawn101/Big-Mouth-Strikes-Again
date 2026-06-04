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

---

## Research Findings (June 4, 2026)

### THE T2 REALITY — Honest Assessment

The iMac Pro 2017 has Apple's FIRST-GENERATION T2 chip. This is currently the **#1 unsolved problem** across ALL OCLP forks:

> *"When these Macs try to boot macOS Tahoe through OpenCore, the T2 chip detects the unauthorized bootloader and triggers a kernel panic — a hard crash before the OS even loads."* — ITECH4MAC

> *"The OCLP team has confirmed this on GitHub Issue #1167 and stated that T2 support requires 'extensive time and research' with no estimate given."*

| Mac Type | Tahoe Status | Via |
|----------|-------------|-----|
| Non-T2 Intel (2012-2017) | ✅ Working | OCLP-Plus v3.2.2 |
| T2 Macs (2017-2019) | ❌ Kernel panic | UNSOLVED by any fork |
| Apple Silicon | ✅ Native | Apple |

**This means our iMac Pro is in the HARDEST category.** But that's exactly why we work on it — nobody else is solving this.

### Forked Repos (8 Total)

| Repo | What It Does |
|------|-------------|
| [OCLP-X](https://github.com/DeadByDawn101/OCLP-X) | JeoJay127's extended Tahoe support (uses max_os) |
| [OCLP-Mod](https://github.com/DeadByDawn101/OCLP-Mod) | laobamac's modified builds (Intel Wi-Fi) |
| [OCLP-Plus](https://github.com/DeadByDawn101/OCLP-Plus) | YBronst's Tahoe patchset v3.2.2 (Broadcom Wi-Fi, Vega Metal) |
| [OpenCore-Legacy-Allow-Tahoe](https://github.com/DeadByDawn101/OpenCore-Legacy-Allow-Tahoe) | TAHOEOCLP fork with root patches |
| [OCLP-lzhoang2801-amfipassbeta](https://github.com/DeadByDawn101/OCLP-lzhoang2801-amfipassbeta) | AMFIPass variant (no amfi=0x80) |
| [OCLP-lzhoang2801](https://github.com/DeadByDawn101/OCLP-lzhoang2801) | Preserved reference OCLP 3.0.0 nightly |
| [OCLP4Hackintosh](https://github.com/DeadByDawn101/OCLP4Hackintosh) | 5T33Z0's Kaby Lake+ guide |
| [lzhoang2801-OCLP](https://github.com/DeadByDawn101/OpenCore-Legacy-Patcher) | Original Tahoe patchset base |

### What's WORKING in OCLP-Plus v3.2.2 (Non-T2 Macs)

- ✅ AMD Vega Metal acceleration (GCN 5 patches)
- ✅ Broadcom Wi-Fi with AWDL (AirDrop works!)
- ✅ AppleHDA audio restoration
- ✅ IOSurface offset patches
- ✅ Metal bundle patches and shims
- ✅ AMFIPass.kext with -amfipassbeta boot arg
- ✅ APFS-only environment handling (macOS 26.4+)
- ✅ KDK integration for audio drivers
- ✅ Works on macOS 26.0 through 26.4.1

### What's BROKEN for T2 Macs (Our iMac Pro)

- ❌ T2 secure boot conflicts with OpenCore bootloader
- ❌ Tahoe's new SIP enforcement causes kernel panic on T2
- ❌ T2 NVMe storage not visible (our Issue #1)
- ❌ T2 audio routing (our Issue #4)

### Revised Strategy

Since T2 is the blocker, we have TWO paths:

**Path A: External Boot (Bypass T2 NVMe)**
1. Install Tahoe to external Thunderbolt 3 SSD
2. Boot from external, bypassing T2 storage entirely
3. Apply OCLP-Plus patches for Vega Metal + Wi-Fi
4. Use as file server + Retina Relay display
5. Internal SSD stays on Sequoia as fallback

**Path B: T2 Kernel Bypass (Research)**
1. Investigate T2 secure boot bypass
2. Study t2linux project (Linux on T2 Macs) for techniques
3. Potentially disable T2 secure boot via Apple Configurator 2
4. Custom kernel extension or SSDT to handle T2 NVMe handshake
5. This is the "nobody has solved it" path

**Recommended: Start with Path A (external boot) for immediate use, research Path B in parallel.**

### Key Technical Notes

- macOS Tahoe = version 26, latest is 26.5 (May 2026)
- Tahoe is the FINAL Intel macOS — macOS 27 will be Apple Silicon only
- Apple is already thinning Intel code from frameworks
- OCLP team is smaller than ever (key contributors left)
- Community forks (OCLP-Plus, OCLP-Mod, OCLP-X) are where active development happens
- AMFIPass.kext + `-amfipassbeta` is better than `amfi=0x80` (more app compatibility)

---

*"We don't give up. We do what others don't and build what isn't possible." — RavenX LLC*

---

## T2 Disable Strategy (June 4, 2026)

### THE KEY INSIGHT

t2linux project ALREADY boots Linux on T2 Macs using ONLY:
1. Startup Security Utility → "No Security"
2. Allow External Boot
3. Custom kernel with apple-bce driver

They DON'T need checkm8! But macOS Tahoe fails because Apple's kernel
EXPECTS a T2 handshake that times out when booting via OpenCore →
AppleKeyStore.kext causes kernel panic.

### THREE-LEVEL APPROACH

**Level 1: Soft Disable (try FIRST — no exploit needed)**
```
1. Boot into Recovery (Cmd+R)
2. Utilities → Startup Security Utility
3. Set "No Security" (disables Intel-side secure boot)
4. Check "Allow booting from external media"
5. Boot macOS Tahoe from external Thunderbolt 3 SSD via OpenCore
6. Block AppleKeyStore.kext in OpenCore Kernel→Block section
7. Apply OCLP-Plus patches for Vega Metal + Wi-Fi + Audio
```

**Level 2: checkra1n T2 Jailbreak (if Level 1 fails)**
```
1. Put T2 in DFU mode (power + specific key combo)
2. Run checkra1n via USB-C to exploit checkm8 vulnerability
3. pongoOS loads → disable T2 secure boot
4. Boot macOS Tahoe normally
5. Semi-tethered: needs re-exploit on each cold boot
```

**Level 3: Permanent T2 Disable Tool (THE COMMUNITY CONTRIBUTION)**
```
1. Use checkra1n + pongoOS to access T2 firmware
2. Write custom pongoOS module that:
   a. Patches T2 secure boot verification routine
   b. Disables AppleKeyStore timeout enforcement
   c. Allows unsigned bootloaders permanently
   d. Preserves T2 hardware functions (NVMe, audio, keyboard)
3. Tool persists across reboots — no re-exploit needed
4. One-time USB-C setup → permanent T2 freedom
```

### WHY THIS IS POSSIBLE

The T2 chip has an UNPATCHABLE vulnerability (checkm8):
- Based on A10 processor (iPhone 7)
- ROM-level exploit — Apple cannot fix via software update
- checkra1n already exploits this for iPhone jailbreaking
- pongoOS already runs custom code on T2
- The toolchain EXISTS — we just need to write the persistence layer

### WHAT WE'D BUILD: `t2-freedom`

```
t2-freedom — Permanent T2 Secure Boot Disable for macOS Tahoe

Usage:
  1. Connect USB-C cable between iMac Pro and another Mac/PC
  2. Put iMac Pro T2 in DFU mode
  3. Run: t2-freedom --disable-secure-boot
  4. T2 secure boot permanently disabled
  5. Boot macOS Tahoe via OpenCore normally

What it does:
  - Exploits checkm8 ROM vulnerability (unpatchable)
  - Loads pongoOS on T2
  - Patches secure boot verification routine
  - Writes persistent config to T2 NVRAM
  - Preserves all T2 hardware functions

What it preserves:
  ✅ NVMe storage access
  ✅ Audio routing (DAC)
  ✅ Keyboard/trackpad (via apple-bce)
  ✅ FaceTime camera
  ❌ Secure boot (intentionally disabled)
  ❌ FileVault encryption (may need reconfiguration)
```

### Forked Repos (12 Total)

#### OCLP Forks (8)
| Repo | Purpose |
|------|---------|
| OCLP-X | Extended Tahoe support |
| OCLP-Mod | Modified builds (Intel Wi-Fi) |
| OCLP-Plus | Tahoe patchset v3.2.2 |
| OpenCore-Legacy-Allow-Tahoe | Root patches |
| OCLP-lzhoang2801-amfipassbeta | AMFIPass variant |
| OCLP-lzhoang2801 | Reference OCLP 3.0.0 nightly |
| OCLP4Hackintosh | Kaby Lake+ guide |
| lzhoang2801-OCLP | Original Tahoe patchset |

#### T2 Research Forks (4)
| Repo | Purpose |
|------|---------|
| [apple-bce-drv](https://github.com/DeadByDawn101/apple-bce-drv) | T2 Bridge/Buffer Copy Engine driver |
| [linux-t2-patches](https://github.com/DeadByDawn101/linux-t2-patches) | Kernel patches for T2 hardware |
| [T2-Ubuntu](https://github.com/DeadByDawn101/T2-Ubuntu) | Ubuntu ISO for T2 Macs |
| [bootra1n](https://github.com/DeadByDawn101/bootra1n) | Minimal Linux for running checkra1n |

### Next Steps

- [ ] **This week:** Set up Retina Relay + file server on current macOS
- [ ] **Level 1 test:** Try "No Security" + external Thunderbolt boot with Tahoe
- [ ] **Level 1 test:** Block AppleKeyStore.kext in OpenCore config
- [ ] **Research:** Study pongoOS source code for persistence mechanisms
- [ ] **Research:** Study apple-bce-drv for T2 communication patterns
- [ ] **Build:** Create `t2-freedom` tool skeleton in Big-Mouth repo
- [ ] **Community:** Post findings to t2linux Discord + OCLP GitHub discussions

---

*"We don't give up. We do what others don't and build what isn't possible." — RavenX LLC*
