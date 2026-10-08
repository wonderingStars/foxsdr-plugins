# Plugin index schema (draft v1)

Served from the plugin repo over HTTPS as `index.json`. The app fetches only
this file until the user explicitly installs something.

```json
{
  "schemaVersion": 1,
  "generated": "2026-08-15T20:00:00Z",
  "plugins": [
    {
      "id": "pocsag-decoder",
      "name": "POCSAG Pager Decoder",
      "version": "1.0.0",
      "author": "FoxSDR project",
      "licence": "MIT",
      "abiVersion": 3,
      "capabilities": ["CASCADE_CAP_DECODER"],
      "summary": "Decodes POCSAG paging transmissions to text.",
      "description": "Longer prose shown in the detail pane.",
      "category": "meters-paging",
      "experimental": false,
      "whatsNew": "1.0.0: first release.",
      "published": "2026-08-15",
      "screenshots": [
        {
          "file": "screenshots/pocsag-decoder/1.png",
          "url": "https://raw.githubusercontent.com/<owner>/<repo>/master/screenshots/pocsag-decoder/1.png",
          "sha256": "<64 hex chars>",
          "sizeBytes": 245000,
          "width": 1040,
          "height": 650,
          "caption": "One line saying what the picture shows."
        }
      ],
      "legalNotice": "Intercepting messages not addressed to you may be unlawful in your jurisdiction (in the UK, Wireless Telegraphy Act 2006 s.48). Install only if you understand your local law.",
      "minSupportedVersion": "1.0.0",
      "homepage": "https://github.com/<owner>/<repo>",
      "platforms": [
        {
          "os": "windows",
          "arch": "x64",
          "file": "pocsag-decoder-1.0.0-abi3-win-x64.dll",
          "url": "https://raw.githubusercontent.com/<owner>/<repo>/master/plugins/pocsag-decoder-1.0.0-abi3-win-x64.dll",
          "sha256": "<64 hex chars>",
          "sizeBytes": 123456
        }
      ]
    }
  ]
}
```

## Field rules the client enforces

- `schemaVersion` must equal 1, else the whole index is refused (a newer
  server must not be reinterpreted by an older client).
- `abiVersion` must equal the host's `CASCADE_PLUGIN_ABI_VERSION` exactly, or
  the entry is shown as "not compatible with this version" and cannot be
  installed. Catching it here saves a download that the loader would refuse.
- `sha256` is MANDATORY. The download is written to a temp file, hashed, and
  only moved into the plugins directory on an exact match. A mismatch is a
  hard failure with the expected and actual digests shown.
- `sizeBytes` is a pre-flight sanity bound; the client also caps any download
  regardless, so a hostile or corrupt server cannot fill the disk.
- `url` must be `https://` and share the index's host, or be on an explicit
  allow-list. Redirects to another host are refused rather than followed.
- `licence` and `legalNotice` are displayed BEFORE the install button is
  enabled. `legalNotice` is optional; when present it must be acknowledged.
- `minSupportedVersion` is the RETIREMENT FLOOR. An installed copy older than
  this is refused by the host, with a message telling the user to update,
  rather than being loaded. It is how a plugin with a known-bad decode or a
  changed output format is taken out of service without waiting for every user
  to notice. Setting it equal to the current version makes that update
  mandatory. Absent means "no floor": any installed version keeps working.
- `capabilities` lists the ABI capability bits the binary declares
  (`CASCADE_CAP_DECODER` for demodulated audio in, `CASCADE_CAP_IQ_DECODER`
  for raw complex baseband). It is generated from the DLL, never hand-written.
  From 0.99.72 the plugin store reads it to say what an uninstalled plugin
  is and what it reaches; once a plugin is fitted, the descriptor the host
  reads from the binary itself is authoritative and wins.

## Store fields (all optional; added for the 0.99.72 plugin store)

`schemaVersion` stays 1. Every client since 0.14 ignores keys it does not
know, so older applications keep working and simply do not show these.

| field | type | meaning |
|---|---|---|
| `category` | string | The shelf the store lists the plugin under: one of `aircraft`, `marine`, `satellites-weather`, `meters-paging`, `broadcast`, `maps-tools`. Absent or unknown lands on the store's OTHER shelf. The generator refuses any other value. |
| `experimental` | bool | `true` for a plugin that has never decoded a real signal. The store draws a badge; the word `EXPERIMENTAL` is no longer part of `name` or the start of `summary`. |
| `whatsNew` | string | One short paragraph about THIS version, starting `"<version>: "` (for example `"1.8.1: Keeps phantom aircraft off the map."`, or `"1.0.0: first release."`). Version history no longer goes in `description`, which stays a plain account of what the plugin does. |
| `published` | string | `YYYY-MM-DD`: the day this version's binary was first committed to the development repository. Computed from git by the generator; omitted (with a warning) when git has no record. |
| `screenshots` | array | Pictures for the plugin's page, in display order; an empty array or an absent key means none. Each is `{ "file", "url", "sha256", "sizeBytes", "width", "height", "caption" }`: a PNG at `screenshots/<id>/<name>.png`, 16:10 within 2 % (1040 x 650 preferred), at most 2 MiB, with a one-line `caption`. The application fetches pictures over `https://` only, asks for nothing but that `url`, and checks `sha256` before it shows one. |

The pictures live in `screenshots/<id>/` in this repository, and the
generator reads `screenshots/<id>/<name>.png` with a one-line
`<name>.txt` beside it for the caption (a picture with no caption, a file that
is not a PNG, a picture that is not 16:10, or one over 2 MiB is an error).

## The catalogue is generated, not edited

`index.json` is produced by `tools/gen_index.py` from a metadata table plus the
DLLs actually sitting in `plugins/`. Do not hand-edit it. The generator:

- computes every `sha256` and `sizeBytes` from the real file, so a hash can
  never drift from the artefact it names;
- **fails** if a metadata row names a binary that is not present — the
  file-less-row failure that once made a hub serve 0-byte firmware to a whole
  fleet;
- **fails** if a binary is present in `plugins/` with no catalogue row, so a
  publish cannot be half-finished;
- **fails** if the version in the row is not part of the filename, since two
  releases sharing a URL would silently overwrite one another;
- runs `tools/probe_plugin.exe` against each DLL to read the descriptor the
  binary *actually* declares, and **fails** on any disagreement over version,
  ABI, licence or author;
- **fails** on a row with no `category` or one outside the six above, a row
  whose `name` or `summary` still carries `EXPERIMENTAL`, a `whatsNew` that
  does not start with the row's version, a version paragraph left in
  `description`, and any picture that breaks the rules above;
- computes `published` from git and reads `screenshots` from the folder, so
  neither is ever typed by hand;
- stamps `generated` with the time the content last changed (an unchanged
  catalogue keeps its stamp, so `--check` stays green).

That last check exists because the catalogue and the binary did disagree the
first time it mattered: the example plugin was republished as 2.0.0 while its
compiled descriptor still said 1.0.0. Nothing but a mechanical comparison
catches that.

Run `py -3.14 tools/gen_index.py --check` in CI: it regenerates in memory and
exits non-zero if the committed `index.json` differs.

## Why the hash is pinned in the index rather than trusting TLS

TLS authenticates the transport, not the artefact. Pinning the digest means a
compromised release asset, a mirror, or a cache cannot substitute a different
DLL — the client refuses anything whose bytes do not match what the index
promised. The remaining trust root is the index itself, which is why it comes
from one fixed HTTPS origin and why signing it is the natural next step.
