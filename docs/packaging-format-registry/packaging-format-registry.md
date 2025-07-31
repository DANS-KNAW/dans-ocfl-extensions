# OCFL Extension: Packaging Format Registry

- **Extension Name:** packaging-format-registry
- **Authors:** Linda Reijnhoudt, Jan van Mansum
- **Minimum OCFL Version:** 1.0
- **Status:** DRAFT

## Overview

In order for an OCFL Repository to be self-contained, it may want to explicitly define and declare the packaging format used for each OCFL Object Version. We
broadly define "packaging format" as a set of rules about the way the content files of an OCFL Object Version are laid out, organized, and described. The
registry is organized in a similar way to [0008-schema-registry](https://ocfl.github.io/extensions/0008-schema-registry.html){:target=_blank}.

## The packaging format registry

Each packaging format is described in a subdirectory of the `packaging_formats` directory in the storage root's extensions directory. These files may include
documentation, examples, machine actionable code and other information about the packaging format. How to interpret these files is outside the scope of this
extension.

## Parameters

Configuration is done by setting values in the file `config.json` at the top level of the extension's directory. The keys expected are:

- Name: `extensionName`
    - Description: String identifying the extension.
    - Type: String
    - Constraints: Must be packaging-format-registry.
    - Default: packaging-format-registry

- Name: `packagingFormatDigestAlgorithm`
    - Description: Algorithm used for calculating safe filenames from packaging formats. Use `name`/`version` as input.
    - Type: String
    - Constraints: Must be a valid digest algorithm returning strings which are safe file names on the target file system.
    - Default: md5

- Name: `digestAlgorithm`
    - Description: Digest algorithm used for calculating fixity of the packaging registry inventory.
    - Type: String
    - Constraints: Must be a valid digest algorithm.
    - Default: The same value used elsewhere in the OCFL for integrity checking.

## packaging_format_inventory.json

A manifest of the registered Packaging Formats must be maintained in`packaging_format_inventory.json`.

- Name: `manifest`
    - Description: Object containing manifest entries.
    - Type: Object
    - Constraints: Must contain one entry per registered Packaging Format; may contain 3 sub-keys documented below. These entry names must be short enough to be
      easily used to identify the packaging format.
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
    - Constraints: The `name` and `version` together must be unique for the packaging formats in this storage root ``
    - Default: Not applicable
- Name: `summary`
    - Description: Short description of the packaging format
    - Type: String
    - Constraints: Not applicable
    - Default: Not applicable

## Implementation

An OCFL Repository with this extension enabled can add extra information for each OCFL object/version that is added to the Repository. For every new OCFL
object/version the Packaging Format for that object/version could be recorded.

To add a new packaging format to the OCFL Root the packaging_format_inventory.json must be updated and the files describing the Packaging Format must be placed
in the `packaging_formats` folder under that subfolder.

A process could be implemented which ensures all OCFL objects/versions written are checked for packaging format references. Where not already registered, the
packaging format must be documented and registered as described below. The packaging format is identified by the name and version referenced. The digest of the
`name`/`version` (as configured by identifierDigestAlgorithm) must be used as a filename. MD5 may be sufficient, other algorithms may be configured.

The values of digestAlgorithm and identifierDigestAlgorithm should not be changed once the registry is initialised. If changing the digest is unavoidable, all
existing entries in the registry must be updated to the new algorithm(s).

How this Packaging Format is recorded, is not specified in this extension. One possible way would be to use the OCFL extension `Object Version Properties`.

When reading an OCFL Object Version the recorded Packaging Format can be consulted for more info on the packaging format of this version.

### Registering a packaging format with the extension

For each referenced packaging format in the OCFL object/versions:

* the entry in the storage_root extension is derived by hashing the `name`/`version` of the packaging format using the configured
  `packagingFormatDigestAlgorithm`.
* the manifest object in the `packaging-format-registry.json` must be checked for this key.
* if this key is not already present in the registry:
* Documentation of the packaging format must be collected, and stored in the packaging format directory with the derived local filename.
* The packaging_format_inventory.json must updated:
* a new key must be created in the manifest object, with the defined properties.
* The packaging_format_inventory.json’s sidecar file must also be updated with the appropriate checksum.

## Example

The packaging_format_inventory.json contains the manifest with the available OCFL Object Packaging Formats for this OCFL Repository.

```
packaging_format_inventory.json
{
  "manifest" : {
    "76f773808534f2969d7a405b99e78b11" : {
      "name" : "BagIt"
      "version" : "v0.97"
      "summary" : "a hierarchical file packaging format for storage and transfer of arbitrary digital content."
    }, 
    "05b408a38e341de9bb4316aa812115ee" : {
      "name" : "BagIt"
      "version" : "v1.0"
      "summary" : "https://datatracker.ietf.org/doc/html/rfc8493"
    }
  }
}
```

The storage root of the OCFL Repository would then look something like this:

```
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
