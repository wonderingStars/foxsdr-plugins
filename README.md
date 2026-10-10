# FoxSDR Plugin Catalogue

The plugin catalogue for **FoxSDR**. The application reads
[`index.json`](index.json) from this repository and offers the plugins it lists
for download from inside the app.

This repository holds **distribution artefacts only** — the catalogue index and
the plugin binaries it names. Nothing here is required to *use* FoxSDR; the
application ships with no plugins and contacts this repository only when you
open the plugin store (the first time in a session) or press its check key,
and, from 0.99.72, when you open a plugin's page, to fetch its pictures.

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

The plugins on offer are the entries of [`index.json`](index.json), which is
always current: each entry carries the plugin's name, version, maker, licence, a
one-line summary, a longer description, what is new in this version, the shelf it
is listed under (`category`), whether it is `experimental`, pictures of it at
work where there are any, and a checksummed download for every platform it is
built for. FoxSDR's plugin store reads that file and lists the plugins by shelf:
aircraft, marine, satellites and weather, meters and paging, broadcast, and maps
and tools. [`SCHEMA.md`](SCHEMA.md) describes every field.

A plugin marked `experimental` has never decoded a real signal. It is published
so that someone who can receive the band can help correct it: do not rely on one
for anything that matters, and in particular not for safety or distress traffic.
Each plugin's own description says what it decodes, what it needs, what has and
has not been tested, and whether a legal notice must be accepted before it
installs.

All are MIT-licensed (each entry states its licence). Most were written
clean-room from published specifications; a plugin's description says how it
was written, and FT8 Decoder, for example, is built on ft8_lib.

## Please read: how far these have been tested

**ADS-B is confirmed working against real aircraft.** A six-second capture at
1090 MHz on a USRP B200 decoded 10 distinct aircraft: 23 positions, 27
velocity reports and 3 callsigns. Two easyJet flights (`EZY595R`, `EZY151Z`)
came back on ICAO addresses in the UK block, and a Ryanair flight (`RYR26QM`)
on an address in the Irish block — and since the address and the callsign
travel in different message types decoded by different code, that agreement is
real evidence rather than a coincidence.

**POCSAG is confirmed on the tested UK signal.** A Windows 1.0.3 candidate
displayed coherent 2400 bit/s alphanumeric messages from a USRP B200
recording at 153.350 MHz and from a live receiver through the FoxSDR 0.99.74
development build on 2026-10-09. The original reported transmission was
unavailable for a before/after comparison, and no received payloads or pager
identities are published here. Version 1.0.4 keeps text as the default and
adds recent decoded messages to its settings page for current released ABI 3
hosts; numeric and function-code modes remain available. Its own page showed
known synthetic function-0 text in the default 520 by 300 window on released
Windows FoxSDR 0.99.72 and 0.99.73, with both app runs exiting successfully.
The final Windows 1.0.4 module also replayed the recorded B200 signal on
released FoxSDR 0.99.73: coherent 2400 bit/s text remained visible in the
default settings window after later tone-only traffic. That replay exited
successfully. Its history keeps eight text/numeric messages and the most
recent tone-only page separately, so tones cannot replace received text.
Automatic flat NFM setup and preserving the tuned frequency require the new
host reception flags, implemented by the 0.99.74 development build.

**The remaining decoders have not yet been confirmed against a real off-air
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
| Privacy | No network access at all unless you open the plugin store; a plugin's pictures (0.99.72 and later) are fetched only when you open that plugin's page, from the URL in the index, carrying nothing else |

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
