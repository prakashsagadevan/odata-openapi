# Tools

Tools for transforming [OData](http://www.odata.org) CSDL (`$metadata`) XML documents into [OpenAPI](https://github.com/OAI/OpenAPI-Specification) documents.

The core ingredient is the [`V4-CSDL-to-OpenAPI.xsl`](V4-CSDL-to-OpenAPI.xsl) transformation. It transforms OData CSDL XML Version 4.0 documents into either OpenAPI 3.0.0 or Swagger 2.0 documents.

OData CSDL XML documents conforming to one of the predecessor OData versions 2.0 or 3.0 can be transformed into OData 4.0 with the [`V2-to-V4-CSDL.xsl`](V2-to-V4-CSDL.xsl) transformation.

The three files [`transform`](transform), [`transform.cmd`](transform.cmd), and [`transform.js`](transform.js) are wrapper scripts for Unix Bash, Windows Command, and Node.js.

## `transform.js` for Node.js

This script transforms one or more OData CSDL (`$metadata`) XML documents into OpenAPI JSON documents. 

It uses [`xslt4node`](https://www.npmjs.com/package/xslt4node), which in turn needs [`node-gyp`](https://www.npmjs.com/package/node-gyp) and a [Java SE JDK](http://jdk.java.net).

### Installation

Install a [Java SE JDK](http://jdk.java.net) and make sure it is in the `PATH`

```sh
javac -version
```

 Install `node-gyp` globally, following the [platform-specific installation instructions for `node-gyp`](https://github.com/nodejs/node-gyp/blob/master/README.md#installation).


Clone or download this repository, go to the `tools` folder and type
```sh
npm install
```

To install globally type
```sh
npm install -g
```
### Usage

Assuming you installed the script globally and your metadata file is `MyMetadata.xml`, then
```sh
odata-openapi -dp MyMetadata.xml
```
will transform it into `MyMetadata.openapi.json` with a nice [yUML](https://yuml.me/) diagram and pretty-printed JSON. 


Just type
```sh
odata-openapi -h
```
to get usage hints
```
Usage: odata-openapi <options> <source files>
Options:
 --basePath              base path (default: /service-root)
 -d, --diagram           include YUML diagram
 -h, --help              show this info
 --host                  host (default: localhost)
 -o, --openapi-version   3.0.0 or 2.0 (default: 3.0.0)
 -p, --pretty            pretty-print JSON result
 -r, --references        include references to other files
 --scheme                scheme (default: http)
```

If you installed the script locally, start it via
```sh
node path_to_tools/transform.js ...
```
(replace `path_to_tools` with your local installation path).


## `transform` for Unix Bash

The main purpose of this script is to refresh all example OpenAPI files in folder [`../examples`](../examples), verify that the produced OpenAPI JSON files conform to the respective JSON schemas, and show diffs to the last committed versions of the JSON files. In other words: it's a simple test tool.

The script transforms all files listed in the control file [`transform.txt`](transform.txt) into both OpenAPI 3.0.0 and Swagger 2.0 documents. The former are suffixed `.openapi3.json`, the latter just `.openapi.json`. 

The control file [`transform.txt`](transform.txt) contains one line per file to be transformed. Each line consists of up to five fields, separated by spaces:
* file name (looked for in `../examples`)
* scheme (`http` or `https`)
* host
* service root path without the trailing slash
* OData version (`V2`, `V3`, or `V4`) (yes, this could be determined from the source file, but I'm too lazy :-)

### Installation

The prerequisites are listed within [`transform`](transform). It's quite a lot:
```sh
sudo apt-get install default-jre
sudo apt-get install git
sudo apt-get install libxalan2-java
sudo apt-get install yajl-tools
sudo apt-get install nodejs
sudo apt-get install nodejs-legacy
sudo apt-get install npm
sudo npm install -g ajv-cli
```
And finally clone https://github.com/OAI/OpenAPI-Specification next to this project:
```sh
git clone https://github.com/OAI/OpenAPI-Specification.git
```
### Usage

In the `tools` folder execute
```sh
./transform
```
to transform all files listed in `transform.txt`.

To transform files not listed in `transform.txt` pass their names as parameters
```sh
./transform path-to-csdl-file ...
```


## `transform.cmd` for Windows Command

The main purpose of this script is to refresh all example OpenAPI files in folder [`../examples`](../examples), verify that the produced OpenAPI JSON files conform to the respective JSON schemas, and show diffs to the last committed versions of the JSON files. In other words: it's a simple test tool. And yes, it's the "original" to `transform` for Unix Bash.

### Installation

The prerequisites are listed within [`transform.cmd`](transform.cmd). It's quite a lot:
- [Java SE 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) is installed and in the `PATH`
- [git](https://git-for-windows.github.io/) is installed and in the `PATH`
- [Xalan](http://xalan.apache.org/xalan-j/downloads.html) is installed and `CLASSPATH` contains `xalan.jar` and `serializer.jar`
- [YAJL](https://github.com/lloyd/yajl)'s `json_reformat` has been compiled and is in the `PATH`
- [Node.js](https://nodejs.org/) is installed
- [ajv-cli](https://www.npmjs.com/package/ajv-cli) is installed
- https://github.com/OAI/OpenAPI-Specification is cloned next to this project

### Usage

In the `tools` folder execute
```sh
transform
```


# Supported Annotations

The [`V4-CSDL-to-OpenAPI.xsl`](V4-CSDL-to-OpenAPI.xsl) transformation can be fine-tuned via annotations in the CSDL (`$metadata`) XML documents.

## [Core](https://github.com/oasis-tcs/odata-vocabularies/blob/master/vocabularies/Org.OData.Core.V1.md)

Term | Annotation Target | OpenAPI field
-----|-------------------|--------------
`Computed` | Property | omit from Create and Update structures
`DefaultNamespace` | Schema | path templates for actions and functions without namespace prefix
`Description` | Action, ActionImport, Function, FunctionImport | `description` of Operation Object
`Description` | EntitySet, Singleton | `description` of Tag Object
`Description` | EntityType | `description` of Request Body Object
`Description` | ComplexType, EntityType, EnumerationType, Parameter, Property, TypeDefinition | `description` of Schema Object
`Description` | Schema, EntityContainer | `info.title`
`Immutable` | Property | omit from Update structure
`LongDescription` | Action, ActionImport, Function, FunctionImport | `description` of Operation Object
`LongDescription` | ComplexType, EntityType, EnumerationType, Parameter, Property, TypeDefinition | `description` of Schema Object
`LongDescription` | Schema, EntityContainer | `info.description`
`SchemaVersion` | Schema | `info.version`


## [Capabilities](https://github.com/oasis-tcs/odata-vocabularies/blob/master/vocabularies/Org.OData.Capabilities.V1.md)

Term | Annotation Target | OpenAPI field
-----|-------------------|--------------
`CountRestrictions/Countable` | EntitySet | `$count` system query option for `GET` operation
`DeleteRestrictions/Deletable` | EntitySet | `DELETE` operation for deleting an existing entity
`ExpandRestrictions/Expandable` | EntitySet, Singleton | `$expand` system query option for `GET` operations
`FilterRestrictions/Filterable` | EntitySet | `$filter` system query option for `GET` operation
`FilterRestrictions/RequiredProperties` | EntitySet | required properties in `$filter` system query option for `GET` operation (parameter description only)
`FilterRestrictions/RequiresFilter` | EntitySet | `$filter` system query option for `GET` operation is `required`
`KeyAsSegmentSupported` | EntityContainer | path templates with key-as-segment convention
`IndexableByKey` | EntitySet | `GET` operation for single entity
`InsertRestrictions/Insertable` | EntitySet | `POST` operation for inserting a new entity
`KeyAsSegmentSupported` | EntityContainer | `paths` URL templates use key-as-segment style instead of parenthesis style
`NavigationRestrictions/RestrictedProperties` <br>&nbsp;&nbsp;&nbsp;`/InsertRestrictions/Insertable` | EntitySet, Singleton | `POST` operation for inserting a new entity
`UpdateRestrictions/Updatable` | EntitySet, Singleton | `PATCH` operation for modifying an existing entity
`SearchRestrictions/Searchable` | EntitySet | `$search` system query option for `GET` operation
`SortRestrictions/NonSortableProperties` | EntitySet | properties not listed in `$orderby` system query option for `GET` operation
`SortRestrictions/Sortable` | EntitySet | `$orderby` system query option for `GET` operation
`SkipSupported` | EntitySet | `$skip` system query option for `GET` operation
`TopSupported` | EntitySet | `$top` system query option for `GET` operation


## [Common](https://wiki.scn.sap.com/wiki/display/EmTech/OData+4.0+Vocabularies+-+SAP+Common)

Term | Annotation Target | OpenAPI field
-----|-------------------|--------------
`FieldControl: Mandatory` | Property | require in Create structure
`Label` | Action, ActionImport, Function, FunctionImport | `title` of Operation Object
`Label` | ComplexType, EntityType, EnumerationType, Property, TypeDefinition | `title` of Schema Object
`QuickInfo` | Action, ActionImport, Function, FunctionImport | `description` of Operation Object
`QuickInfo` | ComplexType, EntityType, EnumerationType, Parameter, Property, TypeDefinition | `description` of Schema Object


## [Validation](https://github.com/oasis-tcs/odata-vocabularies/blob/master/vocabularies/Org.OData.Validation.V1.md)

Term | Annotation Target | OpenAPI field
-----|-------------------|--------------
`AllowedValues` | Property | `enum` of Schema Object - list of allowed (string) values
`Pattern` | Property | `pattern` of Schema Object

