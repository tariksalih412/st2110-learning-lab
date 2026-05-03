# 📡 ST 2110 Broadcast Infrastructure — Interactive Learning Lab

> An interactive, browser-based simulation of a professional SMPTE ST 2110 IP broadcast facility — built as a self-directed learning project to deeply understand modern broadcast networking standards.

!\[License](https://img.shields.io/badge/license-MIT-blue)
!\[Standard](https://img.shields.io/badge/SMPTE-ST%202110-red)
!\[PTP](https://img.shields.io/badge/IEEE-1588%20PTP-yellow)
!\[NMOS](https://img.shields.io/badge/AMWA-NMOS%20IS--04%2F05-green)
!\[Built with](https://img.shields.io/badge/built%20with-HTML%2FJS%20%2B%20AI%20assistance-purple)

\---

## 🎯 What Is This?

This project is a **self-learning tool** I built to understand how modern broadcast facilities work at the IP layer. Rather than just reading specs, I wanted to *see* what happens when:

* A PTP Grandmaster Clock loses GPS lock
* A network spine switch fails mid-broadcast
* ST 2022-7 seamless protection kicks in
* A satellite uplink drops while 480,000 viewers are watching

The result is an interactive simulation covering the full broadcast chain — from camera sources to viewer platforms — with real fault injection and live system response.

\---

## 🚀 Live Demo

Open any of these files directly in your browser — no server needed:

|File|What It Shows|
|-|-|
|`control-room.html`|Production Control Room — Vision Switcher, Multiviewer, VU Meters|
|`smpte2110-noc.html`|Network Operations Center — Spine-Leaf topology, fault injection|
|`tv-channel-sim.html`|Full TV Channel — Studio to viewer, all platforms|
|`smpte2110-diagram-en.html`|ST 2110 Architecture — Annotated technical reference|
|`smpte2110-simulation.html`|Fault Simulator — PTP loss, path failure, dual-path|

\---

## 📚 What I Learned Building This

### SMPTE ST 2110 — IP Media Transport

The standard separates video, audio, and ancillary data into independent RTP/UDP streams (called *essences*), each timestamped against a shared PTP clock. This was the hardest mental shift from SDI thinking — in SDI everything is bundled; in ST 2110 everything travels independently and gets reassembled at the destination.

**Key sub-standards I studied:**

* `ST 2110-10` — System timing and PTP reference model
* `ST 2110-20` — Uncompressed video (RFC 4175 payload)
* `ST 2110-21` — Traffic shaping: NL / N / W sender types
* `ST 2110-22` — Compressed video (JPEG XS)
* `ST 2110-30` — PCM audio (built on AES67)
* `ST 2110-31` — AES3 / Dolby-E passthrough
* `ST 2110-40` — Ancillary data (timecode, captions, tally)

### IEEE 1588 PTP — Precision Time Protocol

The most critical piece I didn't fully appreciate before: every essence stream in ST 2110 carries RTP timestamps referenced to a *shared* PTP clock. Without it, the receiver cannot align separate video and audio streams. A Grandmaster Clock (GPS-disciplined) distributes timing to every device with sub-microsecond accuracy.

When I simulated PTP loss, I understood *why* lip-sync errors happen — it's not the audio being late, it's the absence of a shared reference that lets you know how late.

### AMWA NMOS — Networked Media Open Specifications

The management layer that ST 2110 doesn't define. NMOS handles:

* **IS-04**: Device discovery and registration (REST API)
* **IS-05**: Connection management — telling a receiver which sender to subscribe to
* **IS-07**: Events and tally — the IP replacement for GPIO wiring
* **IS-08**: Audio channel mapping

Before NMOS, every vendor had a proprietary control API. NMOS is what makes multi-vendor ST 2110 facilities actually work.

### ST 2022-7 — Seamless Protection Switching

Send two identical copies of every RTP stream over two physically separate network paths. The receiver compares RTP sequence numbers and picks the best packet from each. When one path fails, the switchover is truly hitless — no frames dropped, no audio glitch. I simulated this and the difference from traditional failover (which always has an interruption) is dramatic.

### Spine-Leaf Network Architecture

A 100GbE production network typically uses:

* **Spine switches** (core, non-blocking) — carry Path A and Path B traffic
* **Leaf switches** (access) — connect to end devices, run PTP Boundary Clocks
* **IGMP snooping** — controls multicast group membership so streams only go where needed

The Bandwidth requirements are significant: a single uncompressed 4K60p stream is \~24 Gbps. A typical facility with 10-20 sources needs a 400G+ fabric.

\---

## 🏗️ Repository Structure

```
st2110-learning-lab/
│
├── simulations/
│   ├── control-room.html          # Full Production Control Room
│   ├── smpte2110-noc.html         # Network Operations Center
│   ├── tv-channel-sim.html        # End-to-end TV Channel
│   ├── smpte2110-simulation.html  # Fault injection simulator
│   └── smpte2110-diagram-en.html  # Architecture reference
│
├── docs/
│   ├── ARCHITECTURE.md            # System design notes
│   ├── STANDARDS.md               # Standards reference
│   └── LEARNING-NOTES.md          # Personal study notes
│
├── README.md
└── LICENSE
```

\---

## 🔧 Key Simulations \& What They Demonstrate

### 1\. PTP Clock Loss

**What happens:** The Grandmaster Clock goes offline. Devices switch to GM-02 (backup) via BMCA. If both GMs fail, every device free-runs on its internal oscillator — drifting at ±50–200 ppm independently. Audio and video desynchronize within milliseconds.

**What I learned:** PTP isn't just nice-to-have — it's the foundation everything else depends on. A facility without redundant Grandmasters is a single point of failure that immediately affects on-air quality.

### 2\. ST 2022-7 Path Failure

**What happens:** SPINE-A (100GbE) fails. The receiver detects an RTP sequence gap on Path A and silently continues using Path B. Zero packets lost. Zero viewer impact.

**What I learned:** The difference between "seamless" and "fast" failover. Traditional redundancy interrupts for 50–200ms while switching. ST 2022-7 is genuinely zero-loss because both paths are always active simultaneously.

### 3\. Encoder Failure

**What happens:** Primary TX encoder crashes. Backup encoder (ENC-2) activates with \~8 seconds of service interruption during ramp-up. Bitrate drops from 48.3 Mbps to 0 then recovers.

**What I learned:** Encoder redundancy is a separate concern from network redundancy. A dual-path network doesn't protect you if the single encoder upstream of the network fails.

### 4\. Satellite Uplink Loss

**What happens:** Uplink carrier drops. Satellite distribution (480,000 viewers) is lost. OTT/IP distribution continues unaffected. CDN absorbs some of the displaced viewers.

**What I learned:** Modern broadcast has layered distribution — satellite, cable, OTT, mobile each have separate failure domains. A complete outage requires failures at multiple independent layers simultaneously.

\---

## 🧠 Technical Details

### Why Pure HTML/JS?

I deliberately chose no frameworks or build tools. This makes the simulations:

* **Instantly runnable** — open in any browser, no npm install
* **Fully readable** — the code is the documentation
* **Easy to fork** — change one number, see immediate effect

Every canvas animation runs in a `requestAnimationFrame` loop. Every fault state is a boolean flag in a global state object. Simple and transparent.

### Canvas-Based Rendering

The broadcast chain topology is rendered on HTML5 Canvas using:

* Bezier curves for signal path visualization
* Animated RTP packet dots along each path
* Real-time glitch effects for PTP drift simulation
* Per-node fault state coloring and shake animation

### Audio Simulation

VU meters and waveforms use `Math.sin()` composites to generate convincing level movement, with peak hold decay and proper segment-by-segment coloring (green → yellow → red at -6 dBFS).

\---

## 📖 Standards Referenced

|Standard|Organization|Description|
|-|-|-|
|SMPTE ST 2110-10/20/21/22/30/31/40|SMPTE|IP media transport suite|
|SMPTE ST 2022-7|SMPTE|Seamless protection switching|
|SMPTE ST 2059-2|SMPTE|PTP profile for broadcast|
|IEEE 1588-2019|IEEE|Precision Time Protocol|
|AMWA IS-04/05/07/08/09/11|AMWA/NMOS|Networked media open specs|
|AES67-2018|AES|Audio over IP interoperability|
|RFC 4175|IETF|RTP payload for uncompressed video|
|RFC 3550|IETF|RTP transport protocol|

\---

## 🗺️ What's Next / Roadmap

Things I want to build or learn next:

* \[ ] Real PTP measurement using `ptpd` or `ptp4l` on Linux
* \[ ] Actual NMOS IS-04 registry using the open-source `nmos-cpp` implementation
* \[ ] Wireshark dissector analysis of real ST 2110 captures
* \[ ] Integration with VLC for actual RTP stream testing
* \[ ] JPEG XS codec exploration (ST 2110-22)
* \[ ] Cloud/REMI workflow simulation (contribution over public internet)

\---

## 🤝 Transparency Note

This project was built as a **learning exercise**, with AI assistance used as a study tool and code accelerator — similar to how one might use Stack Overflow, vendor documentation, or a study group. The underlying standards research, architectural decisions, and understanding of *why* things work the way they do are genuine outcomes of this learning process.

I'm actively studying toward deeper hands-on experience with real ST 2110 equipment.

\---

## 📄 License

MIT — use it, fork it, learn from it.

\---

## 🔗 Resources That Helped Me

* [SMPTE ST 2110 Overview — The Broadcast Bridge](https://thebroadcastbridge.com)
* [AMWA NMOS Specifications](https://specs.amwa.tv/nmos/)
* [EBU Technology \& Innovation — IP Studio](https://tech.ebu.ch/ip-studio)
* [VSF TR-10 Technical Recommendations](https://www.videoservicesforum.org)
* [AES67 \& ST 2110 Audio — Dante / RAVENNA resources](https://www.aes.org)
* [The AIMS Alliance](https://aimsalliance.org)

\---

*Built by Tarik Salih — learning broadcast engineering one standard at a time.*

