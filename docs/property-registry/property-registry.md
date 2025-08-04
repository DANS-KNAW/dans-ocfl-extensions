OCFL Extension: Property Registry
=================================

- **Extension Name:** property-registry
- **Authors:** Linda Reijnhoudt, Jan van Mansum
- **Minimum OCFL Version:** 1.0
- **Status:** DRAFT

Overview
--------

This extension facilitates a way to define properties for an OCFL object. This extension only describes which properties could be recorded, and their
definitions and structures. It does not prescribe how these are stored for objects or object versions. One way to do this is to use
the [Object Version Properties extension](../object-version-properties/object-version-properties.md).

Parameters
----------

Configuration is done by setting values in the file `config.json` at the top level of the extension's directory. The expected key are:

- Name: `extensionName`
    - Description: String identifying the extension.
    - Type: String.
    - Constraints: Must be `property-registry`.
    - Default: `property-registry`.

- Name: `propertyRegistry`
    - Description: An object with as keys the names of the properties, and as values property description entries as defined below.
    - Type: Object.
    - Constraints: The values must have at least the required properties as described below, at most extended with the optional properties.
    - Default: Empty array.

### Property description entry

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
    - Type: Object.
    - Constraints: Each value in the object must be a property description entry as defined above.
    - Required: if `type` is `object`.

- Name: `required`
    - Description: A boolean indicating whether this property is required.
    - Type: Boolean.
    - Constraints: None.
    - Required: false.
    - Default: false.

Note, that the `array` type is not supported, as this would make validation of the property values more complex. The properties defined by means of this
extension are intended to be fairly simple, and not to be used for complex data structures, which are better stored in the content files of the OCFL object.

Note also, that this extension does not define a generic way to validate the property values against the constraints.

Implementation
--------------

### Validation

As part of the validation of the storage root, implementing software must check that the `config.json` is well-formed and contains the required keys and values
as described above. It must also check that no other keys are present in the `config.json` file.

### Registering a property with the property registry

In order to register a property with the property registry, the implementing software must update the `config.json` file in the `property-registry` extension
directory with a new property description entry.

Examples
--------

The following example registry contains three properties, each demonstrating a different use case. The registry is stored in the `property-registry/config.json`
file in the `extensions/property-registry` directory of the OCFL storage root.

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
  "propertyRegistry": {
    "retentionEndDate": {
      "description": "the date until which this Object Version must be retained in this repository",
      "type": "string",
      "constraint": "a duration in ISO 8601 format, e.g. P1Y2M3D",
      "required": true
    },
    "deaccessioned": {
      "description": "If present, this version of the object has been deaccessioned and should not be disseminated",
      "type": "object",
      "constraint": "",
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
          "constraint": "",
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

### `retentionEndDate` property

This is a simple string property that indicates the date until which this object version must be retained in the repository. It is required and must be in ISO
8601 format.

### `deaccessioned` property

This is a structured property that doubles as a boolean flag. If present, it indicates that this object version has been deaccessioned and should not be
disseminated. It also documents the date and reason for deaccessioning.

### `packagingFormat` property

This is a simple string property that indicates the packaging format of the current object version as defined in
the [Packaging Format Registry extension](../packaging-format-registry/packaging-format-registry.md). It is required and must be a valid packaging format name
as defined in that registry.
