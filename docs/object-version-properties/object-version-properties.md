# OCFL Extension: Object Version Properties

- **Extension Name:** object-version-properties
- **Authors:** 
- **Minimum OCFL Version:** 1.0
- **Status:** DRAFT

## Overview
This extension facilitates a way to record metadata on an OCFL object version that is not recorded in the object version itself. This can be necessary because it is related to the repository (e.g. packaging format information) or because it is only available after the object version is already archived. Since OCFL repository objects are immutable, they cannot be changed.
This includes deaccessioning or deletion actions.
This extension only describes how the properties should be recorded, it does not prescribe which properties will be recorded.

## Parameters
The `object_version_properties.json` file in the object_root extensions directory might contain entries for all versions in this object. Per version, it lists keys and a value or object containing subproperties.

## Implementation
Implementing software might check the well-formedness of the `object-version-properties.json` in the extension folder of the object_root.

Implementing software might check whether the file `object-version-properties.json` is available for each OCFL Object and has a key for each version in this object.
(**TODO:** maybe this should not be mandatory?)

## Example
Two examples will be given:

1. a simple property
2. a complex property

### 1. a simple property
if the repository wants to record a simple property, for instance the archival date for each OCFL Object Version, it could achieve that in the following way:

```
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── object-version-properties.md
  ├── 0de
  |   └── 45c
  |       └── f24
  |           └── item1
  │               └── 0=ocfl_object_1.0
  │                   ├── inventory.json
  │                   ├── inventory.json.sha512
  |                   └── extensions
  │                        └── object-version-properties/
  │                            ├── object_version_properties.json
  │                            └── object_version_properties.json.sha512
  └────────────────── v1/
                      ├── inventory.json
                      ├── inventory.json.sha512
                      └── content/
                          └── ... files ...
```
the `object-version-properties.json` of the OCFL object could have the following entry:

```
{
  "v1" : {
    "archival-date" : "2024-07-30T21:12:04"
  }
}
```

### 2. a complex property
if the repository wants to record a complex object for a version, for instance that it has been deaccessioned, this could be done with a similar structure. The `object-version-properties.json` of the OCFL object might have the following entry:

```
{
  "v2": {"archival-date" : "2025-10-15T13:19:00"},
  "v1" : {
    "archival-date" : "2024-07-30T21:12:04"
    "deaccessioned" : {
      "datetime" : "2025-10-15T13:19:00",
      "reason" : "Deaccessioned because dataset was deleted in Easy"
    }
  }
}
```
