# Overview
This extension facilitates a way to record metadata on an OCFL object version that is not recorded in the object version itself. This can be necessary because it is related to the repository (e.g. packaging format information) or because it is only available after the object version is already archived. Since OCFL repository objects are immutable, they cannot be changed.
This includes deaccessioning or deletion actions.
This extension only describes how the properties should be recorded, which properties to use are described in other extensions or in it's `config.json`. 

# Parameters
the `object_version_properties.json` file in every object_root contains entries for all versions in this object. Per version, it contains a list of keys (as declared in the `config.json` of this extension in the storage_root) and a value or object containing the properties defined in . Each property is declared in the `config.json` of the extension, and is described with the following attributes:
`description`, `type`, `constraint`, `properties`, `extension` and `mandatory`. 
Some of these are disjunct. 


|key|description|mandatory|
|---|---|---|
|description|human readable explanation of this property|true|
|type| a json type (number, string, boolean or object). An object needs the `properties` to be filled|true|
|extension| if used, the name MUST be an extension in the storage_root that describes the functionality|false|
|constraint||false|
|mandatory| boolean|true|
|properties| an array of properties with the following keys|false|
|properties.name||true|
|properties.description||true|
|properties.type| a json type (number, string, boolean)|true|
|properties.constraint||false|
|properties.mandatory|boolean whether this property is mandatory within the parent object|true|

# Implementation
Implementing software might check whether the extensions mentioned in the `config.json` are installed in the storage_root, and whether these have a valid `object-version-properties` object in their `config.json`. 

Implementing software might check for each additional object version into the object_root whether the `mandatory` properties are provided in the `object-version-properties.json` for this version. 


# Example
Three examples will be given:

1. a simple property
2. a complex property
3. a complex property that uses another extension

## 1. a simple property
if the repository wants to record a simple property, for instance the archival date for each OCFL Object Version, it could achieve that in the following way:

```
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── object-version-properties.md
  ├── extensions
  |   └── object-version-properties/
  |       └── config.json    
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
with the following content for `object-version-properties/config.json`:

```
{
  "archival-date" : {
      "description" : "The date on which this Object Version has been archived in this repository"
      "type" : "string", 
      "constraint": "a datetime in ISO 8601 YYYY-MM-DDTHH-mm-ss format",
      "mandatory" : true
    },
}
```
and the `object-version-properties.json` of the OCFL object would have the following entry:

```
{
  "v1" : {
    "archival-date" : "2025-10-15T13:19:00"
  }
}
```

## 2. a complex property
if the repository wants to record a complex object for a version, for instance that it has been deaccessioned, this could be done with the same structure, but with a different content for object-version-properties/config.json`:

```
{
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
and the `object-version-properties.json` of the OCFL object might have the following entry:

```
{
  "v1" : {
    "deaccessioned" : {
      "datetime" : "2025-10-15T13:19:00",
      "reason" : "Deaccessioned because dataset was deleted in Easy"
    }
  }
}
```
## 3. a complex property that uses another extension
If the property to be recorded is described in an extension, the `object-version-properties.json` would look like this:

```
{
  "packaging-format" : {
    "description" : "The packaging format of this Object Version",
    "type" : "object",
    "extension" : "packaging-format-registry",
    "mandatory" : true
  }
}
```

the `config.json` of this extension should then describe the properties to be used:

```
packaging-format-registry/config.json
{
  "extensionName" : "packaging-format-registry",
  "version" : "0.1.0",
  "object-version-properties" : {
    "description" : "the packaging format used for this object version",
    "type" : "string",
    "constraint" : "one of the keys in the packaging-format-registry.json",
    "mandatory": true
  }
}
```

## A storage_root with all the above examples implemented

```
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── object-version-properties.md
  ├── packaging-format-registry.md
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
  │               └── ... files describing the packaging_format ...  
```

the config.json in the object-version-properties extension declares which storage-root extensions can be used in the version-properties.json in the object versions.

```
object-version-properties/config.json
{
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
    "properties" [{
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
    }]
  },
  "packaging-format" : {
    "description" : "The packaging format of this Object Version",
    "type" : "object",
    "extension" : "packaging-format-registry",
    "mandatory" : true
  }
}
```

```
packaging-format-registry/config.json
{
  "extensionName" : "packaging-format-registry",
  "version" : "0.1.0",
  "object-version-properties" : {
    "description" : "the packaging format used for this object version",
    "type" : "string",
    "constraint" : "one of the keys in the packaging-format-registry.json",
    "mandatory": true
  }
}

```

and two different possible instantiations of `object_version_properties.json`


```
object_version_properties.json

{
  "v2": {
    "archival-date" : "2020-09-28T16:22:44",
    "packaging-format": "BagIt v0.97"
  },
  "v1": {
    "archival-date" : "2018-03-19T06:22:11",
    "packaging-format": "DANS RDA BagPack v0.1.0",
    "deaccessioned": {
      "datetime": "2020-09-28T13:55:00",
      "reason": "Deaccessioned because dataset was deleted in Easy"
    } 
  }
}
```

```
object_version_properties.json

{
  "v1": {
    "archival-date" : "2002-07-12T15:14:33",
    "packaging-format": "DANS RDA BagPack v0.1.0"    
  }
}
```
