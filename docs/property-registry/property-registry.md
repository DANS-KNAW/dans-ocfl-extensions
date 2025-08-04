OCFL Extension: Property Registry
=================================

- **Extension Name:** property-registry
- **Authors:** Linda Reijnhoudt, Jan van Mansum
- **Minimum OCFL Version:** 1.0
- **Status:** DRAFT

Overview
--------

This extension facilitates a way to define properties for an OCFL object. This extension only describes which properties could be recorded, and their
definitions and structures. It does not prescribe how these are stored for objects or object versions.

Parameters
----------

Configuration is done by setting values in the file `config.json` at the top level of the extension's directory. The keys expected are:

- Name: `extensionName`
    - Description: String identifying the extension.
    - Type: String.
    - Constraints: Must be `property-registry`.
    - Default: `property-registry`.

- Name: `propertyRegistry`
    - Description: An list of property descriptions, each describing a property that can be recorded for an OCFL object or object version.
    - Type: Array of objects.
    - Constraints: The objects must have at least the required properties as described below, at most extended with the optional properties.
    - Default: Empty array.

### Property description entry

- Name: `name`
    - Description: The name of the property.
    - Type: String.
    - Constraints: None.
    - Required: true.
    - Default: None.

- Name: `description`
    - Description: A human-readable description of the property, explaining its purpose and usage.
    - Type: String.
    - Constraints: None.
    - Required: true.
    - Default: None.

- Name: `type`
    - Description: The JSON type of the property.
    - Type: String.
    - Constraints: Must be one of: `number`, `string`, `boolean` or `object`. If `object`, the property must have a `properties` key defined.
    - Required: true.
    - Default: `string`.

- Name: `constraints`
    - Description: A human-readable description of the constraints on the property value, if any.
    - Type: String.
    - Constraints: None.
    - Required: false.

- Name: `properties`
    - Description: If the property is of type `object`, this key must be present and contain an object with as keys the names of the properties of the object,
      and as values property description entries as defined b
    - Type: object.
    - Constraints: Each value in the object must be a property description entry as defined above.
    - Required: if `type` is `object`.

## Implementation

Implementing software might check whether the extensions mentioned in the `config.json` are installed in the storage_root.

Implementing software might check whether the `config.json` is well-formed.

## Example

Three examples will be given:

1. a simple property
2. a complex property
3. a complex property that uses another extension

### 1. a simple property

if the repository wants to record a simple property, for instance the archival date for each OCFL Object Version, it could achieve that in the following way:

```text
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── property-registry.md
  ├── extensions
      └── property-registry/
          └── config.json    
```

with the following content for `property-registry/config.json`:

```json
{
  "extensionName": "property-registry",
  "archival-date": {
    "description": "The date on which this Object Version has been archived in this repository",
    "type": "string",
    "constraint": "a datetime in ISO 8601 YYYY-MM-DDTHH-mm-ss format",
    "mandatory": true
  }
}
```

### 2. a complex property

if the repository wants to record a complex object for a version, for instance that it has been deaccessioned, this could be done with the same structure, but
with a different content for `property-registry/config.json`:

```json
{
  "extensionName": "property-registry",
  "propertyRegistry": [
    {
      "name": "archival-date",
      "description": "If given, this version of the object has been deaccessioned and should not be disseminated",
      "type": "object",
      "constraint"
      ""
      "mandatory": false,
      "properties"
    {
      "name": "datetime"
      "description": "The date on which this object version has been deaccessioned"
      "type": "string",
      "constraint": "a datetime in ISO 8601 YYYY-MM-DDTHH-mm-ss format",
      "mandatory": true
    },
    {
      "name": "reason",
      "description": "The reason why this object version has been deaccessioned.",
      "type": "string",
      "constraint": "",
      "mandatory": true
    }
  ]
}
}
}
```

### 3. a complex property that uses another extension

If the property to be recorded is described in an extension, the `property-registry/config.json` would look like this:

```
{
  "extensionName" : "property-registry",
  "packaging-format" : {
    "description" : "The packaging format of this Object Version",
    "type" : "object",
    "extension" : "packaging-format-registry",
    "mandatory" : true
    "properties" : [{
      "description" : "the packaging format used for this object version",
      "type" : "string",
      "constraint" : "one of the name/version pairs in the packaging-format-registry.json",
      "mandatory": true
    }]
  }
}
```

### A storage_root with all the above examples implemented

```
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── property-registry.md
  ├── packaging-format-registry.md
  ├── extensions
  |   ├── property-registry/
  |   |   └── config.json  
  │   └── packaging-format-registry/
  │       ├── config.json
  |       ├── packaging_format_inventory.json
  |       ├── packaging_format_inventory.json.sha512
  │       └── packaging_formats
  │           ├── 05b408a38e341de9bb4316aa812115ee
  │           |   └── ... files describing the packaging_format ...
  │           └── 76f773808534f2969d7a405b99e78b11
  │               └── ... files describing the packaging_format ...  
```

`property-registry/config.json` would then contain the following json:

```
{
  "extensionName" : "property-registry",
  "archival-date" : {
      "description" : "The date on which this Object Version has been archived in this repository"
      "type" : "string", 
      "constraint": "a datetime in ISO 8601 YYYY-MM-DDTHH-mm-ss format",
      "mandatory" : true
  },
  "deaccessioned" : {
    "description" : "If given, this version of the object has been deaccessioned and should not be disseminated",
    "type" : "object",
    "constraint": "",
    "mandatory" : false,
    "properties" [
      {
        "name" : "datetime",
        "description" : "The date on which this object version has been deaccessioned",
        "type" : "string", 
        "constraint": "a datetime in ISO 8601 YYYY-MM-DDTHH-mm-ss format",
        "mandatory" : true
      }, {
        "name" : "reason",
        "description": "The reason why this object version has been deaccessioned.",
        "type" : "string",
        "constraint" : "",
        "mandatory": true
      }
    ]
  },
  "packaging-format" : {
    "description" : "The packaging format of this Object Version",
    "type" : "object",
    "extension" : "packaging-format-registry",
    "mandatory" : true,
    "properties" : [{
      "description" : "the packaging format used for this object version",
      "type" : "string",
      "constraint" : "one of the name/version pairs defined in the packaging-format-registry.json",
      "mandatory": true
    }]
  }
}
```
