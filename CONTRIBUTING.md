# Contributing to CCC Control Catalogs

## Overview

This repository contains the control catalogs for the [FINOS Common Cloud Controls](https://www.finos.org/common-cloud-controls-project) project. Each `controls.yaml` file conforms to the Gemara [`ControlCatalog`](https://github.com/gemaraproj/gemara) schema — a standardized, machine-readable data model for GRC engineering.

At release time, the [CCC delivery toolkit](https://github.com/finos/common-cloud-controls/tree/main/delivery-toolkit) ingests these files using the [`go-gemara`](https://github.com/gemaraproj/go-gemara) Go SDK and converts them to Markdown artifacts for publication.

## Repository Structure

Controls are organised by service domain and type:

```
<domain>/<service-type>/controls.yaml
```

For example:

```
storage/object/controls.yaml
crypto/key/controls.yaml
compute/virtual-machines/controls.yaml
```

## Schema

Two top-level keys should be present in every artifact: `controls` and `imports`.

### `controls`

A list of controls native to this service type. Each entry requires:

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique identifier in the form `CCC.<ServiceCode>.CN<NN>` |
| `title` | string | Short label for the control |
| `objective` | string | Unified statement of intent for this control |
| `assessment-requirements` | list | Requirements that confirm the control objective has been met |

Each `assessment-requirements` entry requires:

| Field | Type | Description |
|---|---|---|
| `id` | string | Identifier in the form `CCC.<ServiceCode>.CN<NN>.AR<NN>` |
| `text` | string | The specific requirement to be verified |
| `applicability` | list | TLP classification levels this requirement applies to |

Example:

```yaml
controls:
  - id: CCC.ObjStor.CN01
    title: Prevent Requests to Buckets or Objects with Untrusted KMS Keys
    objective: |
      Prevent any requests to object storage buckets or objects using
      untrusted KMS keys to protect against unauthorized data encryption,
      or sensitive data decryption.
    assessment-requirements:
      - id: CCC.ObjStor.CN01.AR01
        text: |
          When a request is made to read a bucket, the service MUST prevent
          any request using KMS keys not listed as trusted by the organization.
        applicability:
          - tlp-amber
          - tlp-red
        recommendation: ""
```

### `imports`

An optional list of references to controls defined in [core-catalog](https://github.com/common-cloud-controls/core-catalog) that also apply to this service type. Each entry requires a `reference-id` pointing to the source catalog and a list of `entries`, each with a `reference-id` and a `remarks` field summarising applicability.

```yaml
imports:
  - reference-id: CCC
    entries:
      - reference-id: CCC.Core.CN01
        remarks: Prevent Unencrypted Requests
      - reference-id: CCC.Core.CN02
        remarks: Ensure Data Encryption at Rest for All Stored Data
```

> **Note:** Catalog-level `metadata` (title, version, publication date, etc.) is not authored here — it is populated automatically at release time by the delivery toolkit.

## Adding a New Control

1. Locate the relevant `controls.yaml` for the service type (e.g. `storage/object/controls.yaml`).
2. Append a new entry to the `controls` list. Assign the next available `CN` number for that service code.
3. Ensure `id`, `title`, `objective`, and at least one `assessment-requirements` entry are all present.
4. If the control addresses a general cloud platform concern already described in [core-catalog](https://github.com/common-cloud-controls/core-catalog), add a reference under `imports` instead of duplicating the definition.

## Adding a New Service Type

1. Create a new directory under the appropriate domain (e.g. `networking/cdn/`).
2. Add a `controls.yaml` file with a `controls` list. Follow the ID convention `CCC.<ServiceCode>.CN<NN>`, choosing a unique service code.
3. Add any applicable core control references under `imports`.
4. Open a pull request — the catalog title, metadata, and group assignments are resolved by the delivery toolkit and do not need to be authored manually.

## Validation

Controls can be validated locally against the Gemara CUE schema using the [CUE CLI](https://cuelang.org/docs/install/):

```sh
cue vet --schema '#ControlCatalog' github.com/gemaraproj/gemara <path>/controls.yaml
```

## References

- [Gemara schema](https://github.com/gemaraproj/gemara) — CUE schema definitions including `ControlCatalog`
- [go-gemara](https://github.com/gemaraproj/go-gemara) — Go SDK used by the delivery toolkit to parse and convert catalogs
- [gemara.openssf.org](https://gemara.openssf.org) — full model documentation
