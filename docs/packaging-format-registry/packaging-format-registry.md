# OCFL Extension: Packaging Format Registry

- **Extension Name:** packaging-format-registry
- **Authors:** Linda Reijnhoudt, Jan van Mansum
- **Minimum OCFL Version:** 1.0
- **Status:** DRAFT

## Overview

In order for an OCFL Repository to be self-contained, it may want to explicitly define and declare the packaging format used for each OCFL Object Version. We
broadly define "packaging format" as a set of rules about the way the content files of an OCFL Object Version are laid out, organized, and described. The
registry is organized in a similar way to [0008-schema-registry](https://ocfl.github.io/extensions/0008-schema-registry.html){:target=_blank}.

We describe a standard layout for the packaging format registry, where this extension stores the packaging formats referenced throughout the OCFL Repository.
This packaging format registry must be implemented by creating and maintaining the following items:

* A `config.json` file containing the configuration parameters for the extension.
* A `packaging_formats` directory containing subdirectories for each packaging format.
* A `packaging_format_inventory.json` file providing an index for the stored packaging formats.
* A sidecar file `packaging_format_inventory.json.sha512` (or other configured digest) containing the digest of the `packaging_format_inventory.json` file, in  
  the same manner as the OCFL inventory files.

## Parameters

Configuration is done by setting values in the file `config.json` at the top level of the extension's directory. The keys expected are:

- Name: `extensionName`
    - Description: String identifying the extension.
    - Type: String
    - Constraints: Must be packaging-format-registry.
    - Default: packaging-format-registry

- Name: `packagingFormatDigestAlgorithm`
    - Description: Algorithm used for calculating safe filenames from packaging formats.
    - Type: String
    - Constraints: Must be a valid digest algorithm returning strings which are safe file names on the target file system.
    - Default: md5

- Name: `digestAlgorithm`
    - Description: Digest algorithm used for calculating fixity of the packaging registry inventory.
    - Type: String
    - Constraints: Must be a valid digest algorithm.
    - Default: The same value used elsewhere in the OCFL for integrity checking.

## The packaging_formats directory

The `packaging_formats` directory is located in the top directory of the `packaging-format-registry` extension. Each packaging format is described in a
subdirectory of this `packaging_formats` directory. The name of this subdirectory must be the hexadecimal serialization of the digest of the string:
`<name>/<version>`, where `<name>` and `<version>` are the name and version of the packaging format as defined in the `packaging_format_inventory.json` (see
below). The digest algorithm used to calculate this digest is defined in the `config.json` file under the key `packagingFormatDigestAlgorithm`.

The subdirectory may include documentation, examples, machine actionable code and other information about the packaging format. How to interpret these files is
outside the scope of this extension. The subdirectory may include other subdirectories, recursively.

## packaging_format_inventory.json

A manifest of the registered Packaging Formats must be maintained in `packaging_format_inventory.json`.

- Name: `manifest`
    - Description: Object containing manifest entries.
    - Type: Object
    - Constraints: Must contain one entry per registered packaging format; must contain 3 sub-keys documented below.
    - Default: Not applicable

### manifest entry properties for `packaging_format_inventory.json`

- Name: `name`
    - Description: The human-readable name of the packaging format.
    - Type: String
    - Constraints: Not applicable
    - Default: Not applicable
- Name: `version`
    - Description: Version of this packaging format.
    - Type: String
    - Constraints: The `name` and `version` together must be unique for the packaging formats in this storage root.
    - Default: Not applicable
- Name: `summary`
    - Description: Short description of the packaging format
    - Type: String
    - Constraints: Not applicable
    - Default: Not applicable

## Implementation

### Validation

Clients that implement this extension must validate the following as part of the validation of the OCFL storage root:

* JSON files must be well-formed JSON, conform to described schemas.
* Manifest entries must correspond one-to-one to the packaging formats in the `packaging_formats` directory.
* Name/version pairs must be unique across the manifest.
* Digest algorithms must be one of the algorithms supported by the OCFL specification.

### Registering a packaging format with the extension

For each referenced packaging format in the OCFL object versions:

* the entry in the storage_root extension is derived by hashing the `name`/`version` of the packaging format using the configured
  `packagingFormatDigestAlgorithm`.
* the manifest object in the `packaging-format-registry.json` must be checked for this key.
* if this key is not already present in the registry:
* Documentation of the packaging format must be collected, and stored in the packaging format directory with the derived local filename.
* The packaging_format_inventory.json must be updated:
* a new key must be created in the manifest object, with the defined properties.
* The packaging_format_inventory.json’s sidecar file must also be updated with the appropriate checksum.

To add a new packaging format to the OCFL Root the `packaging_format_inventory.json` file must be updated and the files describing the Packaging Format must be
placed in the `packaging_formats` folder under that subfolder.

The values of digestAlgorithm and packagingFormatDigestAlgorithm should not be changed once the registry is initialized. If changing the digest is unavoidable,
all existing entries in the registry must be updated to the new algorithm(s).

### Referencing a packaging format in an OCFL object version

An implementation can determine the packaging format used for an OCFL object version by finding the name and version of the packaging format used. How this is
recorded is outside the scope of this extension, but one possible way is to use
the [DANS Object Version Properties extension](../object-version-properties/object-version-properties.md) to record the packaging format as a property of the
object version. Other possibilities include using a designated version level metadata field, or a specific file in the object version content directory.

## Example

The packaging_format_inventory.json contains the manifest with the available OCFL Object Packaging Formats for this OCFL Repository.

**packaging_format_inventory.json**

```json
{
  "manifest": {
    "76f773808534f2969d7a405b99e78b11": {
      "name": "BagIt",
      "version": "v0.97",
      "summary": "a hierarchical file packaging format for storage and transfer of arbitrary digital content."
    }, 
    "05b408a38e341de9bb4316aa812115ee": {
      "name": "BagIt",
      "version": "v1.0",
      "summary": "https://datatracker.ietf.org/doc/html/rfc8493"
    }
  }
}
```

The storage root of the OCFL repository would then look something like this:

```text
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  └── extensions
      └── packaging-format-registry/
          ├── config.json
          ├── packaging_format_inventory.json
          ├── packaging_format_inventory.json.sha512
          └── packaging_formats
              ├── 05b408a38e341de9bb4316aa812115ee
              |   ├── rfc8493.txt
              |   └── ... files describing the packaging_format BagIt/v1.0 ...
              └── 76f773808534f2969d7a405b99e78b11                  
                  └── ... files describing the packaging_format BagIt/v0.97 ...  
  
```
