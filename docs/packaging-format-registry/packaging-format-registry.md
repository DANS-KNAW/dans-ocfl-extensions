# OCFL Extension: Packaging Format Registry

- **Extension Name:** packaging-format-registry
- **Authors:**
- **Minimum OCFL Version:** 1.1
- **Status:** DRAFT

# Overview

In order for an OCFL Repository to be self-contained, it may want to explicitly define and declare the packaging format used for each OCFL Object Version. We
broadly define "packaging format" as a set of rules about the way the content files of an OCFL Object Version are laid out, organized, and described. We
describe the packaging format registry as a directory containing subdirectories, each of which describes a specific packaging format.

# Parameters

## packaging_format_inventory.json

A manifest of the registered Packaging Formats must be maintained in`packaging_format_inventory.json`.

- Name:`manifest`
    - Description: Object containing manifest entries.
    - Type: Object
    - Constraints: Must contain one entry per registered Packaging Format; may contain 3 sub-keys documented below. These entry names must be short enough to be
      easily used in the version_packaging_format.json
    - Default: Not applicable

### manifest entry properties for `packaging_format_inventory.json`

- Name:`name`
    - Description: The human-readable name of the packaging format.
    - Type: String
    - Constraints: Not applicable
    - Default: Not applicable
- Name:`version`
    - Description: Version of this packaging format.
    - Type: String
    - Constraints: The `name` and `version` together must be unique for the packaging formats in this storage root ``
    - Default: Not applicable
- Name:`summary`
    - Description: Short description of the packaging format
    - Type: String
    - Constraints: Not applicable
    - Default: Not applicable

# Implementation

An OCFL Repository with this extension enabled must add extra information for each OCFL Object Version that is added to the Repository. For every new OCFL
Object/Version the Packaging Format for that Object/Version must be recorded.

To add a new packaging format to the OCFL Root the packaging_format_inventory.json must be updated and the files describing the Packaging Format must be placed
in the `packaging_formats` folder under that subfolder.

How this Packaging Format is recorded, is not specified in this extension. One possible way would be to use the OCFL extension `Object Version Properties`.

When reading an OCFL Object Version the recorded Packaging Format can be consulted for more info on the packaging format of this version.

# Example

The packaging_format_inventory.json contains the manifest with the available OCFL Object Packaging Formats for this OCFL Repository.

**[storage_root]/extensions/packaging-format-registry/packaging_format_inventory.json**

```json
{
    "manifest" : {
        "40cdd53d9a263e5466b8954d82d23daa" : {
            "name" : "BagIt",
            "version" : "v0.97",
            "summary" : ""
        }, 
        "95d751340dcdc784fd759dbc7ddb9633" : {
            "name" : "BagIt",
            "version" : "v1.0",
            "summary" : "https://datatracker.ietf.org/doc/html/rfc8493"
        }
    }
}
```

In this example we use the Object Version Properties Extension to record the Packaging Format used for each version in this OCFL Object

**[object_root]/extensions/object-version-properties/object_version_properties.json**

```json
{
    "v1" : {
      "packaging-format": "40cdd53d9a263e5466b8954d82d23daa"
    }
}
```

The complete OCFL Repository would then look something like this:

```text
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── extensions
  |   ├── object-version-properties/
  |   |   └── config.json
  │   └── packaging-format-registry/
  │       ├── config.json
  |       ├── packaging_format_inventory.json
  |       ├── packaging_format_inventory.json.sha512
  │       └── packaging_formats
  │           ├── 40cdd53d9a263e5466b8954d82d23daa
  │               └── ... files describing the packaging_format ...
  │           └── 95d751340dcdc784fd759dbc7ddb9633
  |               ├── rfc8493.txt
  │               └── ... files describing the packaging_format ...  
  ├── 0de
  |   └── 45c
  |       └── f24
  |           └── item1
  │               └── 0=ocfl_object_1.0
  │                   ├── inventory.json
  │                   ├── inventory.json.sha512
  |                   └── extensions
  │                       └── object-version-properties/
  │                           ├── object_version_properties.json
  │                           └── object_version_properties.json.sha512
  └────────────────── v1/
                      ├── inventory.json
                      ├── inventory.json.sha512
                      └── content/
                          └── ... files ...
  
```

# our use case

```
packaging_format_inventory.json
{
    "manifest" : {
        "6619641f-6504-447f-9c63-1cfc71815f97" : {
            "name" : "DANS RDA BagPack"
            "version" : "v1"
            "summary" : "A BagIt-based Packaging Format defined by the RDA, with additional rules specified by DANS"
        }
    }
}
```

```
version_packaging_format.json
{
    "v1" : "6619641f-6504-447f-9c63-1cfc71815f97"
}
```

Placing this packaging-format.md file in the storage_root of an OCFL repository allows for this extension to be used in the extensions directory of the
storage_root and object_root.

```
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── object-version-registry.md
  ├── packaging-format-registry.md 
  ├── extensions/
  |   ├── object-version-properties/
  |   |   └── config.json
  │   └── packaging-format-registry/
  │       ├── config.json
  |       ├── packaging_format_inventory.json
  |       ├── packaging_format_inventory.json.sha512
  │       └── packaging_formats
  │           └── 6619641f-6504-447f-9c63-1cfc71815f97
  |               ├── RDA bagpack DANS profile v1.json 
  |               ├── RDA bagpack DANS profile v1.md  
  │               └── Research Data Repository Interoperability WG Final Recommendations.pdf
  ├── 0de
  |   └── 45c
  |       └── f24
  |           └── item1
  │               └── 0=ocfl_object_1.0
  │                   ├── inventory.json
  │                   ├── inventory.json.sha512
  |                   └── extensions/
  │                       └── object-version-properties/
  │                           ├── object_version_properties.json
  │                           └── object_version_properties.json.sha512
  └────────────────── v1/
                      ├── inventory.json
                      ├── inventory.json.sha512
                      └── content/
                          └── ... files ...
  
```
