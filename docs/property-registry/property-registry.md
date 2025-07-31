# OCFL Extension: Property Registry

- **Extension Name:** property-registry
- **Authors:** Linda Reijnhoudt, Jan van Mansum
- **Minimum OCFL Version:** 1.0
- **Status:** DRAFT

## Overview
This extension facilitates a way to define properties for an OCFL object.
This extension only describes which properties could be recorded, and their definitions and structures. It does not prescribe how these are stored for objects or object versions. 

## Parameters
The `config.json` in this extension has the key `extensionName`. Furthermore it defines the properties using the following keys: `name`, 
`description`, `type`, `constraint`, `properties`, `extension` and `mandatory`. 


|key|description|mandatory|
|---|---|---|
|description|human readable explanation of this property|true|
|type| a json type (number, string, boolean or object). An object needs the `properties` to be filled|true|
|extension| if used, the name MUST be an extension in the storage_root that describes the functionality|false|
|constraint||false|
|mandatory| boolean|true|
|properties| an array of properties with the following keys|false|
|property.name||true|
|property.description||true|
|property.type| a json type (number, string, boolean)|true|
|property.constraint||false|
|property.mandatory|boolean whether this property is mandatory within the parent object|true|

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

```
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

```
{
  "extensionName" : "property-registry",
  "archival-date" : {
      "description" : "The date on which this Object Version has been archived in this repository"
      "type" : "string", 
      "constraint": "a datetime in ISO 8601 YYYY-MM-DDTHH-mm-ss format",
      "mandatory" : true
    },
}
```


### 2. a complex property
if the repository wants to record a complex object for a version, for instance that it has been deaccessioned, this could be done with the same structure, but with a different content for `property-registry/config.json`:

```
{
  "extensionName" : "property-registry",
  "deaccessioned" : {
    "description" : "If given, this version of the object has been deaccessioned and should not be disseminated",
    "type" : "object",
    "constraint" ""
    "mandatory" : false,
    "properties" [{
      "name" : "datetime"
      "description" : "The date on which this object version has been deaccessioned"
      "type" : "string", 
      "constraint": "a datetime in ISO 8601 YYYY-MM-DDTHH-mm-ss format",
      "mandatory" : true
    }, {
      "name" : "reason",
      "description": "The reason why this object version has been deaccessioned.",
      "type" : "string",
      "constraint" : "",
      "mandatory": true
    }]
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
