# CCC Control Catalogs

This repository contains the control catalogs for the [FINOS Common Cloud Controls](https://www.finos.org/common-cloud-controls-project) project — machine-readable definitions of the security controls applicable to each cloud service type.

Controls describe the safeguards that should be in place to mitigate the threats defined in [threat-catalogs](https://github.com/common-cloud-controls/threat-catalogs), grounded in the capabilities defined in [capability-catalogs](https://github.com/common-cloud-controls/capability-catalogs).

## Repository Structure

Catalogs are organised by service domain and type:

```
<domain>/<service-type>/controls.yaml
```

For example:

```
storage/object/controls.yaml
compute/virtual-machines/controls.yaml
crypto/key/controls.yaml
```

Cross-cutting controls that apply to all service types are maintained separately in [core-catalog](https://github.com/common-cloud-controls/core-catalog) and referenced here via the `imports` key.

## Delivery

At release time, the [CCC delivery toolkit](https://github.com/finos/common-cloud-controls/tree/main/delivery-toolkit) ingests these files and produces versioned Markdown artifacts for publication.

## License

[Community Specification License 1.0](LICENSE)
