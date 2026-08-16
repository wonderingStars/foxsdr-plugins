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
      "abiVersion": 2,
      "capabilities": ["CASCADE_CAP_DECODER"],
      "summary": "Decodes POCSAG paging transmissions to text.",
      "description": "Longer prose shown in the detail pane.",
      "legalNotice": "Intercepting messages not addressed to you may be unlawful in your jurisdiction (in the UK, Wireless Telegraphy Act 2006 s.48). Install only if you understand your local law.",
      "minSupportedVersion": "1.0.0",
      "homepage": "https://github.com/<owner>/<repo>",
      "platforms": [
        {
          "os": "windows",
          "arch": "x64",
          "file": "pocsag-decoder-1.0.0-win-x64.dll",
          "url": "https://raw.githubusercontent.com/<owner>/<repo>/master/plugins/pocsag-decoder-1.0.0-win-x64.dll",
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
  for raw complex baseband). It is informational for the UI — the host reads
  the authoritative value from the binary itself — and it is generated from
  the DLL, never hand-written.

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
  ABI, licence or author.

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
