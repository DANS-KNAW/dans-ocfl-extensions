Collaboration of these three OCFL Extensions
============================================

This document gives an example of a way to combine the three OCFL extensions to achieve a way to prescribe the properties to be recorded and how to record them
for each OCFL object version.

Repository layout
-----------------
In this example, the OCFL repository is structured as follows:

```text
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── object-version-properties.md
  ├── packaging-format-registry.md
  ├── property-registry.md
  ├── extensions
  |   ├── property-registry/
  |   |   └── config.json  
  │   └── packaging-format-registry/
  │       ├── config.json
  |       ├── packaging_format_inventory.json
  |       ├── packaging_format_inventory.json.sha512
  │       └── packaging_formats
  │           ├── 05b408a38e341de9bb4316aa812115ee 
  │           │    └── ... files describing the packaging_format BagIt/v1.0 ...
  │           ├── 76f773808534f2969d7a405b99e78b11 
  │           │    └── ... files describing the packaging_format BagIt/v0.97 ...
  │           └── 15e5e7397258f296a04219bce1defdff
  │                └── ... files describing the packaging_format DANS BagPack Profile/v1.0.0 
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
  ├────────────────── v1/
  │                   ├── inventory.json
  │                   ├── inventory.json.sha512
  │                   └── content/
  │                        └── ... files ...
  │
  ├────────────────── v2/
  │                   ├── inventory.json
  │                   ├── inventory.json.sha512
  │                   └── content/
  │                        └── ... files ...
```

Content of the extensions
-------------------------

### Property registry

The `config.json` file in the property-registry extension declares the properties that can be used in the `object_version_properties.json` file of an OCFL
object version.

```json
{
  "extensionName": "property-registry",
  "propertyRegistry": {
    "retentionEndDate": {
      "description": "the date until which this object version may be retained in this repository",
      "type": "string",
      "constraint": "a date in ISO 8601 format, e.g. 2030-10-01",
      "required": true
    },
    "deaccessioned": {
      "description": "If present, this version of the object has been deaccessioned and should not be disseminated",
      "type": "object",
      "required": false,
      "properties": [
        {
          "name": "datetime",
          "description": "The date on which this object version has been deaccessioned",
          "type": "string",
          "constraint": "a datetime in ISO 8601 YYYY-MM-DDTHH-mm-ss format",
          "required": true
        },
        {
          "name": "reason",
          "description": "The reason why this object version has been deaccessioned.",
          "type": "string",
          "required": true
        }
      ]
    }
  },
  "packagingFormat": {
    "description": "The packaging format of the current object version, as defined in the packaging-format-registry",
    "type": "string",
    "required": true
  }
}
```

### Packaging format registry

The `config.json` file in the packaging-format-registry extension defines how to document the available OCFL Object Packaging Formats.

```json
{
  "extensionName": "packaging-format-registry",
  "digestAlgorithm": "sha512",
  "packagingFormatDigestAlgorithm": "md5"
}
```

The `packaging_format_inventory.json` file in the packaging-format-registry extension contains the manifest with the available OCFL Object Packaging Formats for
this OCFL Repository.

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
    },
    "15e5e7397258f296a04219bce1defdff": {
      "name": "DANS BagPack Profile",
      "version": "v1.0.0",
      "summary": "A DANS specific profile for the BagPack packaging format."
    }
  }
}
```

The names of the manifest entries are the digests of `name` + `/` + `version` of the packaging format, using the `digestAlgorithm` defined in the `config.json`,
e.g.,

```commandline
echo -n "BagIt/v1.0" | md5sum # Output: 05b408a38e341de9bb4316aa812115ee
```

The files documenting the packaging formats are stored in the subdirectories with the same names.

### Object version properties

If the object in the example repository had two versions the `object_version_properties.json` file might look like this:

```json
{
  "v2": {
    "retentionEndDate": "2071-03-18",
    "packaging-format": "DANS BagPack Profile/v1.0.0"
  },
  "v1": {
    "retentionEndDate": "2071-03-18",
    "packaging-format": "DANS BagPack Profile/v1.0.0",
    "deaccessioned": {
      "datetime": "2025-12-31T13:55:00",
      "reason": "Deaccessioned because confidential data was removed"
    }
  }
}
```

Both versions have a `retentionEndDate` of `2071-03-18`, which is the date until which this object version may be retained in this repository. Both are packaged
using the `DANS BagPack Profile/v1.0.0` packaging format, which is defined in the packaging-format-registry extension. The first version was deaccessioned on
`2025-12-31T13:55:00` because confidential data was removed, and should not be disseminated.


