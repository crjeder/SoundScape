# Procedural Soundscape Engine (5.1 → Denon AVR)

Generate a procedurally-created ambient soundscape (e.g. a bar: background chatter, clinking ice cubes, a piano player in the corner) that plays for hours without a noticeable loop, routed to a 5.1 surround system via an AV receiver.

## Concept

- **Bed layer**: continuous background (room tone, murmur of voices) as long, cross-faded loops with variation.
- **Event layer**: individual sound sources (ice cubes, piano, conversation snippets) as spatial objects, triggered probabilistically rather than on a fixed loop.
- **Anti-loop techniques**: granular synthesis / sample randomization (pitch, timing, volume), Markov chains or Poisson processes for event timing, generative MIDI for the piano instead of a looped audio file.
- **Spatialization**: Ambisonics panning decoded directly to 5.1 channels (L/R/C/LFE/SL/SR) — the same "objects with coordinates" principle Dolby Atmos uses, but self-rendered instead of relying on a licensed decoder.

## Bill of Materials

| Component | Option A (Budget) | Option B (Recommended) | Option C (Most Reliable) |
|---|---|---|---|
| **Compute** | Raspberry Pi 4 (4GB) | Raspberry Pi 5 (8GB) | Mini-PC (Intel N100) |
| **Audio output** | Pi HDMI direct | Pi HDMI direct | USB audio interface (Behringer UMC404HD, 4 out) |
| **Channel routing** | HDMI → Denon (6ch PCM, can be unstable) | HDMI → Denon (6ch PCM) | USB → RCA/XLR → Denon Multichannel-In |
| **Storage** | microSD 32GB | microSD 64GB (A2 rated) | NVMe SSD via USB |
| **Software** | Pure Data | SuperCollider | SuperCollider |
| **Cooling** | Passive | Active (fan case) | Passive (Mini-PC) |
| **Est. cost** | ~€60 | ~€90 | ~€180–250 |

**Notes:**
- Option A: cheapest entry point; HDMI 6-channel PCM can be flaky depending on driver/kernel version.
- Option B: good balance — modern `vc4-hdmi` drivers (Bullseye/Bookworm onward) are noticeably more stable for multichannel PCM than on the Pi 3.
- Option C: bypasses HDMI multichannel issues entirely via dedicated audio hardware — most reliable, at extra cost.
- If your Denon AVR has a **6× RCA multichannel analog input**, Option C is the safest path.

## Setup Guide

### 1. Choose your compute + output path
Pick from the BOM table above based on budget vs. reliability needs.

### 2. Raspberry Pi HDMI multichannel setup (Options A/B)
1. Use Raspberry Pi OS (Bullseye or newer — `vc4-hdmi` driver).
2. Enable full HDMI drive in `/boot/config.txt`:
   ```
   hdmi_drive=2
   ```
3. Configure ALSA for 6-channel output via `~/.asoundrc` (channel mapping to L/R/C/LFE/SL/SR).
4. **Test before building anything on top:**
   ```
   speaker-test -c 6 -D hw:0
   ```
   Confirm all 6 channels arrive correctly at the Denon before proceeding.

### 3. USB audio interface setup (Option C)
1. Connect interface (e.g. Behringer UMC404HD) via USB.
2. Wire outputs to Denon's Multichannel Analog In (RCA).
3. Configure as default ALSA/JACK device — no HDMI driver dependency.

### 4. Install audio engine
- **Pure Data (Pd)**: lighter weight, simpler patching, `ambipan` for spatialization.
- **SuperCollider**: more powerful for generative/algorithmic composition, `PanAz` for spatialization, better suited for Markov-chain-driven event generation and generative MIDI.

### 5. Build the soundscape
1. Create the **bed**: long ambient loop(s), cross-faded, with subtle pitch/timing randomization.
2. Create **event objects**: individual samples (ice clinking, chatter snippets) assigned a spatial position, triggered via a Poisson process or similar probabilistic scheduler — not a fixed interval.
3. Generate the **piano part** algorithmically (e.g. Markov chain over a scale/chord progression) rather than looping a fixed audio file, to avoid perceptible repetition over long sessions.
4. Route all objects through the Ambisonics encoder/decoder to the 6 output channels.

## Open-Source Tools Reference

| Purpose | Tool |
|---|---|
| Procedural audio synthesis | SuperCollider, Pure Data, Csound |
| Ambisonics panning/decoding | SPARTA plugins, Blender Ambisonics tools, built-in Pd/SC objects |
| Object-based audio metadata (open standard) | ADM (Audio Definition Model) |

*Note: Dolby Atmos itself is proprietary — there is no open-source encoder producing certified Atmos bitstreams. This setup achieves the same object-based spatial principle through open tools, self-rendered directly to your 5.1 speaker layout instead of going through a licensed Atmos decoder.*
