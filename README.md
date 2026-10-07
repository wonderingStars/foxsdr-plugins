# FoxSDR Plugin Catalogue

The plugin catalogue for **FoxSDR**. The application reads
[`index.json`](index.json) from this repository and offers the plugins it lists
for download from inside the app.

This repository holds **distribution artefacts only** — the catalogue index and
the plugin binaries it names. Nothing here is required to *use* FoxSDR; the
application ships with no plugins and contacts this repository only when you
open the plugin browser and press Browse.

> **These plugins need FoxSDR 0.14.0 or newer.** They are built against plugin
> ABI 3, and the host requires an exact ABI match. On an older FoxSDR every
> entry here will show as incompatible and cannot be installed — update the
> application and they become available again. If you already have plugins
> installed from before, updating FoxSDR disables them until you press Update
> on each; the rebuilt versions are already published here and waiting.
>
> ABI 3 is intended to be the last such break. Capabilities are now additive,
> so a future decoder type will not retire anything.

## Available plugins

| Plugin | What it decodes | Needs | Notes |
|---|---|---|---|
| **ADS-B** 1.0.0 | Aircraft position, callsign, altitude, velocity at 1090 MHz | Raw I/Q, ≥2 MS/s (2.4 recommended) | ✅ **Verified against real aircraft.** Airborne messages; surface position not yet parsed |
| **AIS** 1.0.0 | Ship identity, position, course, voyage data | Raw I/Q, 192 kS/s | Decodes both marine channels at once; emits `!AIVDM` |
| **APRS / AX.25** 1.0.0 | Amateur packet radio, 144.800 MHz | NFM audio | Mic-E not yet parsed into fields |
| **POCSAG** 1.0.0 | Pager messages, all three bit rates | NFM audio | **Legal notice must be accepted before install** |
| **EAS / SAME** 1.0.0 | US Emergency Alert System and NOAA Weather Radio alert headers | NFM audio | Seven NOAA Weather Radio presets; **not an alerting device** — notice must be accepted |
| **Inmarsat-C / EGC** 0.1.0 | SafetyNET maritime broadcasts | Raw I/Q, 24 kS/s | ⚠ **EXPERIMENTAL — has never decoded a real signal** |
| **DMR Monitor** 0.1.1 | DMR signalling only: talkgroups, radio IDs, colour code, emergency/encrypted flags. No voice | NFM audio, 12.5 kHz DMR channel | ⚠ **EXPERIMENTAL — not yet confirmed against a real off-air burst.** No software voice decoding (patent-encumbered vocoder); no hardware dongle support in this release either. **Legal notice must be accepted before install** |
| **TETRA Monitor (data only, no voice)** 0.1.2 | TETRA cell identity, system information, clear messages. **Plays no audio**: no voice, no decryption | Raw I/Q, 18 ksymbol/s; receiver sample rate 40 kS/s to 9.072 MS/s | ⚠ **Verified only against a synthetic transmitter — not yet decoded a real cell off air.** **Legal notice must be accepted before install.** 0.1.2 fixes a crash at very high sample rates (30.72 MS/s was reported) and refuses any receiver rate outside that range, saying why. **Windows and Linux builds.** The Linux 0.1.2 module was built and tested on GitHub's own runners on 2026-10-07; a Linux copy of 0.1.1 is offered the update |
| **Demod Analyzer (EXPERIMENTAL)** 0.1.0 | Nothing is decoded to a message; it ANALYSES a digitally modulated signal — BPSK, QPSK, 8PSK or 16-QAM with a confidence, symbol rate, carrier, roll-off, EVM, MER, SNR, timing/phase jitter, IQ imbalance, and the Gray-mapped bits | Raw I/Q, any rate: the live receiver's VFO channel, or a cs8/cu8/cs16/cf32/WAV recording it reads itself | ⚠ **EXPERIMENTAL — not yet tried against a real off-air signal.** One combined dashboard picture; the file path is typed, there is no file picker |
| **Radiosondes (EXPERIMENTAL)** 0.1.0 | Vaisala RS41 weather-balloon sondes on 400–406 MHz: serial, position, height, climb rate and wind, with a trail on the map; up to four at once. No temperature, humidity or pressure, no landing prediction | Raw I/Q, any rate from about 20 kS/s (2.4 MS/s suggested) | ⚠ **EXPERIMENTAL — the block CRC and Reed-Solomon layout have not yet been confirmed on frames from a real flight, so it may decode nothing until then.** **Windows build only so far** |
| **Meteor-M LRPT (EXPERIMENTAL)** 0.1.0 | Meteor-M N2-3 / N2-4 weather-satellite pictures on 137.100 and 137.900 MHz: three image channels side by side, building line by line through a pass | Raw I/Q, any rate from 144 kS/s (1.024 MS/s suggested) | ⚠ **EXPERIMENTAL — has never decoded a real signal, and on a real pass will not draw a picture.** The radio, error-correction and packet layers follow published CCSDS documents; the picture layer is a reconstruction, so on a real pass it should lock and name the packets, then say plainly that it cannot draw the image. **Windows build only so far** |
| **VDL Mode 2 (EXPERIMENTAL)** 0.1.0 | The VHF airband digital data link: ACARS messages (registration, label, flight, text) and ground-station XID frames (position, airports, frequencies); every other frame is named with its length, never dropped | Raw I/Q, any rate from 32 kS/s to 61.44 MS/s; up to twelve channels of 136.700–136.975 MHz at once | ⚠ **EXPERIMENTAL — not yet decoded a real aeroplane.** Checked only against a transmitter written from the same reading of ICAO Annex 10 and ETSI EN 301 841; fifteen details could not be confirmed from a document, and if one is wrong it decodes nothing. No CPDLC or ADS-C. **Legal notice must be accepted before install.** **Windows build only so far** |
| **HFDL (EXPERIMENTAL)** 0.1.0 | HFDL, the HF half of airliners' ACARS: ground-station squitters, log-ons, ACARS messages and aircraft position reports, with an instrument window and the positions on the map | USB audio, 12 kHz; a 3 kHz channel | ⚠ **EXPERIMENTAL — has never decoded a real signal and will most likely decode nothing until corrected against a capture.** About sixty values below the published outline of the physical layer are reconstructions, and the self-test uses a transmitter built from the same table, so it proves agreement and nothing more. Map positions come from an unconfirmed encoding. **Legal notice must be accepted before install.** **Windows build only so far** |
| **NAVTEX & DSC (marine)** 1.0.0 | NAVTEX maritime safety warnings (518, 490 and 4209.5 kHz) printed whole, and Digital Selective Calling on VHF channel 70 and the HF distress and safety frequencies: format, category, sender MMSI, nature of distress, position and UTC; calls with a position are plotted as vessels | USB audio (VHF: NFM), 12 kHz | ⚠ **Never tried on a real signal:** the DSC calls in the test are built from the message tables, so they prove agreement with the test transmitter, not with a ship. **Not a GMDSS watch receiver — nothing sounds an alarm — and a distress alert decoded here is real.** Tune 1.7 kHz below the published frequency in USB (the presets do). **Legal notice must be accepted before install.** **Windows build only so far** |
| **Wireless M-Bus meters (EXPERIMENTAL)** 0.1.0 | European utility meters at 868 MHz, modes S, T and C: manufacturer, meter number, kind, signal level and data records; AES-128 modes 5 and 7 are decrypted only for meters whose key you list in the plugin's one setting | Raw I/Q, any rate from 250 kS/s to 61.44 MS/s (2.4 MS/s suggested) | ⚠ **EXPERIMENTAL — never decoded a real meter.** Decodes every complete OMS Annex N example telegram exactly, but the 3-out-of-6 mapping, the Manchester and NRZ bit conventions and frame format B (paid EN 13757-4) are unverified. **Legal notice must be accepted before install.** **Windows build only so far** |
| **Example RMS Reporter** 2.0.0 | Nothing; reports audio level | NFM audio | Reference plugin and template |

All are MIT-licensed and were written clean-room from published
specifications. No third-party decoder implementation was consulted in writing
any of them.

## Please read: how far these have been tested

**ADS-B is confirmed working against real aircraft.** A six-second capture at
1090 MHz on a USRP B200 decoded 10 distinct aircraft: 23 positions, 27
velocity reports and 3 callsigns. Two easyJet flights (`EZY595R`, `EZY151Z`)
came back on ICAO addresses in the UK block, and a Ryanair flight (`RYR26QM`)
on an address in the Irish block — and since the address and the callsign
travel in different message types decoded by different code, that agreement is
real evidence rather than a coincidence.

**The other decoders have not yet been confirmed against a real off-air
signal.** Each is validated against a test transmitter written from the same
reading of the specification. That demonstrates the two halves agree with each
other; it does not prove either is right about the standard.

Where published reference data exists it has been used as an independent check,
and those checks are real:

- **ADS-B** decodes published Mode S reference frames to their published
  positions and callsigns.
- **AIS** transmits a published `!AIVDM` reference sentence through its own
  receiver and reproduces it character for character.
- **POCSAG** derives its BCH polynomial independently, then confirms the
  specification's own synchronisation and idle codewords have zero syndrome
  under it.
- **APRS** matches the published CRC-16/X.25 check value `0x906E`.
- **EAS / SAME** takes its event codes, originator codes and field rules
  from 47 CFR 11.31 itself, and its state numbers from the Census Bureau
  file, so the tables are anchored to the sources the transmitters are
  built to. The demodulation is not: no off-air alert has been decoded.

**Inmarsat-C has no such check available, and is the one to be careful with.**
About ten constants of its air interface could not be confirmed against any
published source and were reconstructed. If any is wrong the plugin decodes
nothing, and that is the likely outcome. It is published at 0.1.0, marked
EXPERIMENTAL, in the hope that someone who can receive the band will supply the
capture needed to correct it. Do not rely on it for anything, and in particular
do not rely on it for safety or distress traffic.

If you get any of these working — or not working — against real signals, that
is the single most useful thing you can report.

## Legal notices

Some decoders here demodulate transmissions whose interception is restricted in
some countries. In the UK, intercepting a message you are not authorised to
receive is an offence under the Wireless Telegraphy Act 2006, s.48.

**EAS / SAME is different, and its notice is not about interception.** Those broadcasts are meant to be received by anyone. The warning is that the plugin is NOT an alerting device: it decodes only what the receiver happens to be tuned to, knows nothing about where you are, and sounds no alarm. Use a certified NOAA Weather Radio receiver for warnings you rely on. Note also that 47 CFR 11.45 forbids transmitting the EAS codes or attention signal, or a recording of them, outside a real emergency or an authorised test - so do not put what you decode back on the air.

Where that applies, the entry carries a `legalNotice` which the application
displays and which you must accept before the Install button is enabled. It is
the author telling you something you need to know, not boilerplate.
**Installing a plugin is your decision and your responsibility.**

## Security model

An application that downloads and loads native code is a code-execution path,
so the client is deliberately strict:

| Control | Behaviour |
|---|---|
| Transport | HTTPS only, certificate validation enabled |
| Integrity | `sha256` is **mandatory** in the index and verified before install |
| Staging | Downloaded to a `.part` file; renamed into `plugins/` only on an exact digest match |
| Redirects | Cross-host redirects are refused, not followed |
| Size | Capped at 64 MiB regardless of what the index claims |
| Filenames | Strict allow-list; `..`, path separators and device names are rejected |
| ABI | `abiVersion` must match the host exactly; mismatches cannot be installed |
| Consent | Nothing installs automatically; the licence is shown first |
| Privacy | No network access at all unless you open the plugin browser |

TLS authenticates the transport, not the artefact. Pinning the digest in the
index means a compromised mirror or a cache cannot substitute a different DLL.

Note that `abiVersion` must match **exactly**, not "at least". When the plugin
ABI changes, every existing plugin stops loading until it is rebuilt. That is
deliberate — the alternative is a struct-layout mismatch that corrupts memory
somewhere unrelated days later — and the application tells you which plugins are
affected and offers the update.

## Index format

See [`SCHEMA.md`](SCHEMA.md).

`index.json` is **generated**, not hand-written: the digests and sizes are
computed from the binaries in `plugins/`, and each binary is loaded and its
compiled descriptor cross-checked against its catalogue entry before publishing.
Editing it by hand here will be overwritten.

## Submitting a plugin

1. Build against `plugin_abi.h` from the FoxSDR source. That header is
   MIT-licensed on purpose: you need no licence from us to write a plugin,
   including a commercial one, and your plugin keeps whatever licence you
   choose.
2. Open an issue here with your DLL, its source, and the metadata for the entry
   (id, name, version, author, licence, summary, description, and a legal
   notice if one applies). The catalogue is regenerated by the maintainers from
   the binary you supply, so the published digest always matches the file that
   was actually reviewed.
3. State your licence honestly. It is shown to every user before they install,
   and again in the application's Plugins panel after loading.

## Licence

The catalogue metadata in this repository is MIT. **Each plugin carries its own
licence**, stated in its index entry — listing a plugin here is not a grant of
any right to it. FoxSDR itself is licensed separately (PolyForm Noncommercial
1.0.0: free for noncommercial use, a paid licence for commercial use); see its
own repository.
