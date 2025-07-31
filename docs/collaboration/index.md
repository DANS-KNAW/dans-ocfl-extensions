# Collaboration of these three OCFL Extensions

This document describes a possible way to combine the three OCFL Extensions to achieve a way to prescribe the properties to be recorded and how to record them for each OCFL Object Version.

```
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
  │               └── ... files describing the packaging_format ...
  │           └── 76f773808534f2969d7a405b99e78b11 
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

the config.json in the property-registry extension declares which storage-root extensions might be used in the object-version-properties.json in the object versions.


```
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
    "mandatory" : true,
    "properties" : [{
      "description" : "the packaging format used for this object version",
      "type" : "string",
      "constraint" : "a name/version pair defined in the packaging-format-registry.json",
      "mandatory": true
    }]
  }
}
```


and two examples of different possible instantiations of `object_version_properties.json`


```
{
  "v2": {
    "archival-date" : "2020-09-28T16:22:44",
    "packaging-format": "BagIt/v0.97"
  },
  "v1": {
    "archival-date" : "2018-03-19T06:22:11",
    "packaging-format": "BagIt/v0.97",
    "deaccessioned": {
      "datetime": "2020-09-28T13:55:00",
      "reason": "Deaccessioned because dataset was deleted in Easy"
    } 
  }
}
```
another example of `object_version_properties.json`:

```
{
  "v1": {
    "archival-date" : "2002-07-12T15:14:33",
    "packaging-format": "BagIt/v1.0"    
  }
}
```

