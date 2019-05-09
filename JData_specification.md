 JData: a general-purpose data storage and interchange format
============================================================

- **Status of This Document**: *This document is current under the state of "Request for Comments". 
It does not specify a standard of any kind.*
- **Copyright Notice**: Copyright (C) Qianqian Fang (2015-2019).
- **License**: Apache License, Version 2.0
- **Version**: RFC.pre.0
- **Abstract**:

JData is a general-purpose data interchange format. It was derived
from the JavaScript Object Notation (JSON) [RFC4627] and Universal 
Binary JSON specifications (Draft 12). JData defines a list of dedicated "name" 
fields to represent a wide range of data structures, including 
scalars, arrays, tables, structures, hashes, links, trees and graphs.


Introduction
------------

### Background


Data are the digital representations of our world. Generating and processing 
data are the essential parts of our daily lives, and are at the very 
foundations of modern sciences, information technologies, businesses, and the 
interactions between our global societies.

Data take many different forms. Some data can be represented by simple scalars; 
some others have complex forms with hierarchical structures. An efficient 
representation of data also strongly depends on the application needs. In some 
cases, plain text files with white-space separated fields are sufficient; 
however, for performance-sensitive applications, using binary formats can 
significantly reduce loading and processing time. The abilities to store and 
parse complex data structures are particularly important to the scientific 
communities.

It is a challenging task to encapsulate wide varieties of data forms in a 
single data interchange format. There has been many previous efforts in 
designing a general-purpose data storage specification. Some of them have 
become popular choices in one or multiple categories of applications. 
Extensible Markup Language (XML), for example, is ubiquitously used as a 
data-exchange format, but the verbosity of the syntax, moderate complexity for 
parsing, impeded readability and inefficiency in expressing structured data 
suggested room for alternative formats. Comma Separated Value (CSV), a rather 
simple plain-text format, is used among some applications to exchange a tabular 
data structure (such as a spreadsheet); yet, its inability to encode more 
complex data forms, lack of flexibility and precision restrict it to specific 
applications.

Hierarchical Data Format (HDF) is a format targeting at the broad needs from 
the scientific communities. It has an extensible hierarchical data model with a 
large capacity to represent complex binary data. However, to effectively use 
HDF require skillful implementation and in-depth understanding to the 
underlying data models. For small projects with non-critical performance needs, 
using and advanced data format such as HDF may requires additional development 
and maintenance efforts. Similar arguments can be made to the Common Data 
Format (CDF) or Network Common Data Format (netCDF) that are partly derived 
from HDF. In addition, the MATLAB mat-file format and Tecplot data format are 
also used among the research communities. Because of the requirement of 
propitiatory software or libraries, these formats have also found difficulties 
to achieve wide-spread use outside of the user communities of the associated 
software.

### JSON and UBJSON

The JavaScript Object Notation (JSON) format is a text-based data format that 
is known for its capability of storing complex data, excellent portability and 
human-readability. JSON is widely adopted in modern web applications, and is 
becoming popular among local/native applications. The key advantages of JSON 
include:

* **simplicity**: JSON data are composed by a list of `"name":value` pairs; 
such a simple grammar greatly eases the use and parsing of the data file; free 
JSON-encoders and decoders are widely available for most popular programming 
languages;
* **human readability**: the text-based nature of JSON and it's clean, 
easy-to-read format make it intuitively readable without in-depth knowledge of 
the format itself;
* **hierarchical data support**: JSON has a tree-like data storage paradigm 
which has the native capacity to support complex hierarchical data structures; 
there is no inherent data size limit imposed by the format itself;
*  **web ready**: because JSON can be readily parsed by JavaScript, most 
JSON-encoded data file can be directly invoked (inline or load from remote 
site) from a JavaScript based web-application.

JSON also has limitations. JSON's "value" fields are weakly-typed. They only 
support strings, numbers, and Boolean types, but lack of the fine-granularity 
to represent various numerical types of different byte-lengths (in C-language, 
for example, short, int, long int, float, double, long double) and their signs 
(signed and unsigned). Because JSON is a text-based format, the size of the 
data file can be significantly larger than a binary format and requires 
additional conversion when used in an application. This introduces overhead in 
both storage and processing.

The Universal Binary JSON (UBJSON) is a binary counterpart to the JSON format. 
It specifically addresses the above mentioned limitations, yet, adheres to a 
simple grammar similar to the text-based JSON. As a trade-off, it does lose the 
"human readability" to a certain extend. Although the implemented parsers for 
UBJSON are not as abundant as JSON, due to the simplicity of the format itself, 
the development cost for implementing a parser for a new programming language 
is significantly lower than other more complex data formats.

With the ease-of-use, superior portability and parser availability, JSON and 
UBJSON have the potentials to serve as the main-stream data storage and 
interchange formats for general needs, especially for the scientific 
communities. A combination of JSON and it's binary counterpart offers features 
that are not currently available with the existing data storage schemes. Yet 
they do not provide all the advanced features found in the more sophisticated 
formats, their greatly simplified encoding and decoding strategies permit 
efficient data sharing among the general audiences.

### JData Specification Summary

JData is a specification for storing, exchanging and processing general-purpose 
data that are commonly encountered in information technology industries and 
research communities. It has a text/UNICODE format derived from the JSON 
specification and a binary format derived from the UBJSON specification. JData 
is designed to represent the commonly used data structures including arrays, 
structures, trees and graphs. A round-trip conversion is defined between the 
text and binary versions of JData documents.

The purpose of this document is to define the text and binary JData format 
specifications. This is achieved through the definition of a semantic layer 
over the JSON/UBJSON data storage syntax to map various types of complex data 
structures. Such semantic layer includes

- a list of reserved "name" fields, or keywords, that define the containers 
of various data types that are commonly used in research,
- a list of reserved "name" fields and formats to facilitate the grouping and 
organization of hierarchical data,
- a list of format properties for the associated "value" field to store the 
specific metadata of the data points, and, in addition,
- a set of conversion rules between the text and binary forms.

In the following sections, we will define the basic JData grammar, data models, 
followed by the keywords for data grouping and various data types, including 
scalars, 1,2,3 and N-dimensional arrays, sparse and complex arrays, structures, 
hashes/associative arrays, trees and graphs. The expressions of these data 
structures in both text and binary forms are specified and exemplified, and 
their conversion rules are defined.

Grammar
------------------------

### Text-based JData Storage Grammar

The text-based JData grammar is identical to the JSON grammar defined in 
[RFC4627], except that JData also accepts the Concatenated JSON Object 
Collection (CJOC), also known as the Concatenated JSON streaming format, in the 
following form:
```
    {
       "object1":{}
    }
    {
       "object2":{}
    }
    ...
```

In CJOC, multiple JSON objects can be included inside a single file or stream, 
separated by 0 or multiple permitted white spaces, namely

`LF (U+000A), CR (U+000D), tab (U+0009), or space (U+0020)`

In the case where 
[JSONReference](http://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) is 
used, the document must have only one root object, and must not contain an CJOC.

### Binary JData Storage Grammar

The binary JData grammar is identical to the UBJSON grammar defined in 
[ubjson.org (Draft 12)](http://ubjson.org), with the array grammar extension to 
support N-dimensional arrays:

`[[] [$] [type] [#] [[] [$] [an integer type] [nx ny nz ...] []] [nx*ny*nz*...*sizeof(type) ] []]`

where the integer type can be one of the UBJSON integer types (i,U,I,l,L), and 
nx, ny, and nz ... are all non-negative numbers specifying the dimensions of 
the N-dimensional array. The binary data of the N-dimensional array is then 
serialized in the **column-major** format (similar to MATLAB or FORTRAN) order.

As a special note, all UBJSON integer types must be stored in the Big-Endian 
format, according to the specification; the storage to the floating point types 
(d,D) follows the IEEE 754 specification.

Data Models
----------------------

### Topological model

Topologically, we define different parts of a JData document using the below nomenclatures

* **"leaflet"** : an element with no child
* **"compound leaf"**, or **"leaf"**: a multi-element data unit completely made of leaflets, or empty
* **"branch"**: a multi-element data unit made of both leaves and leaflets.
* **node**: a leaflet, a leaf or a branch
* **composite node**: a leaf or a branch
* **root**: the top-most node of a JSON object
* **super-root**: the top level node in a JData document containing multiple 
JSON/UBJSON objects in the form of an CJOC
* **named node**: a node in the form of `"name":{...}` or `"name":[]`, in the case 
of named and index leaves and branches, respectively, or `"name":value` in the case
of a leaflet
* **index node**: a node in the form of `{...}` or `[]`, in the case of named 
and index leaves and branches, respectively, or a single `value` in the case
of a leaflet
* **a structure**: a named or index node made of named sub-nodes, represented by `{...}`, 
a structure can be empty
* **an array**: a named or index node made of index sub-nodes, represented by `[...]`,
an array can be empty

A leaflet can only take one of the two possible forms: `value`, referred to as the
"index leaflet", and a `"name": value` pair, referred to as the "named leaflet". 
Here `value` should not contain any sub-element.

The topological elements of a JData document - "leaflet", "leaf", "branch", "root" 
and "super-root" - have specific meanings according to the definitions above.

### Semantic model

To more efficiently process the represented data, we can semantically annotate the 
underying data stucture and organize them for finer granularity. The data grouping 
annotations are

* **"data record"**: a "meaningful" datum, either in the form of a simple data point 
(leaflet) or a complex structure (a composite node); a data record can be a leaflet, 
a leaf, or a collection of leaflets or leaves. 
* **"dataset"**: a set of "logically connected" data records, such as the citation 
information of a paper
* **"data group"**: a group of datasets of "logical connections" 
* **"auxiliary data"**: nodes that store "weakly-relevant" or "irrelevant" data to 
the processing. the auxiliary data can appear at any level in the data annotation hierarchy.

The interpretations to the "meaningful datum", "logically connected data", and 
"weakly-relevant or irrelevant data" are application dependent.

The data organization annotations are ranked in the below order, from the highest 
level to the lowest level:

` super-root > root >=  data group >= dataset >= data record >= leaf > leaflet `

for each annotation level, it can only contain elements annotated of the same 
level or lower levels, but not the upper level annotations. All data annotation
objects can be empty, i.e. do not contain any element. An example JData 
organization schematic is shown below

```
    super-root{
        root1{
            group1{
                dataset1{
                    dataset1.1{...}
                    ...
                    ... other auxiliary data ...
                }
                dataset2{...}
            },
            group2{
                dataset3{...}
            }
	        group3{}
            dataset4{...}
            ... other auxiliary data ...
        }
        root2{A data object that does not have meta-data
            ...
        }
    }
```
A data group, dataset or data record can be topologically presented as a branch 
(including root) or a leaf, but not a leaflet.


Data Annotation Keywords
------------------------

All JData keywords are case sensitive. Data groups, datasets and data records 
can contain metadata to include additional information regarding the data, 
such as name, create date and user-defined headers. However, metadata can 
also present in any branch or leaf.


### Data Group Keywords

The use of data grouping keywords is not mandatory in a JData document. 
Nonetheless, properly partitioning and grouping the data based on their semantic 
relationships and name the components accordingly can greatly enhance the 
portability and readability of the data, and thus, are strongly recommended.

#### Data Group

A data group can be stored in the form of a named node, either 

` "_DataGroup_": { ... }`

or

` "_DataGroup_": [ ... ]`

A data group annotation can have a name in the below form

` "_DataGroup_(a name string): {}`

or

` "_DataGroup_(a name string): []`

If multiple data groups are stored inside a structure, a unique name 
string must be provided for each data group. For example

```
  {
      "_DataGroup_(name1)":{...},
      "_DataGroup_(name2)":{...},
      "_DataGroup_(name3)":[...],
      ...
  }
```
When defining one of the elements of an array object as a data group, one 
must convert that index node into a structure. For example, in the below
array
```
  [
      indexed_node1, indexed_node2, indexed_node3,...
  ]
```
if one needs to define the `indexed_node2` into a data group, one must define
```
  [
      indexed_node1,
      {
          "_DataGroup_(name1)": [index_node2]
      },
      index_node3,
      ...
  ]
```
where the name is optional.

#### Dataset

Similarly, a dataset is specified by

`"_Dataset_": { ... }`

or

` "_Dataset_": [ ... ]`

with an optional name parameter in the following format:

`"_Dataset_(a name string)": ...`

####  Data record

Also similarly, a dataset is specified by

`"_DataRecord_": { ... }`

or

` "_DataRecord_": [ ... ]`

with an optional name parameter in the following format:

`"_DataRecord_(a name string)"`


### Metadata

Metadata records can be associated with a data group, dataset or data record
(or any branch or leaf). It is optional. Metadata can have two forms

* **inline metadata**: metadata defined as part of the annotation tag string, and
* **metadata node**: a metadata structure added as the first element of a node (named or indexed)

The inline metadata is defined by attaching a series of comma-separated `property=value` 
strings to the data group keywords (_DataGroup_,_Dataset_,_DataRecord_), separated by "::".
For example

`"_{DataGroup,Dataset,DataRecord}_(name)::Property1=...,Property2=...,..."`

The `property` name should not contain space or comma. If comma appears inside the `value`,
it must be escaped using the form `\,`. 

An inline metadata can be attached to any named node, for example

`"name::Property1=...,Property2=...,..."`

One or more above mentioned white spaces may be inserted before or after separators "::", "," 
and "=".

The inline metadata is primarily used to store simple properties. When storage of more complex 
metadata is needed, a dedicated "metadata node" shall be inserted to the data object as the
first element of the annotated object. 

For example, if the data to be annotated is a structure, the metadata node is defined 
by a named leaf with a specific keyword "\_MetaData\_". An example can be 
found below:

```
  {
    "_MetaData_": {
       "Property1": "...",
       "Property2": "...",
       "Property3": "...",
       ...
    },
    named_node1,
    named_node2,
    ...
  }
```
If the data to be annotated is an array, the metadata record is an indexed leaf as

```
  [
    {
       "_MetaData_": {
         "Property1": "...",
         "Property2": "...",
         "Property3": "...",
         ...
       },
    }
    indexed_node1,
    indexed_node2,
    ...
  ]
```

The only required sub-field of the meta-data is "\_Name\_". All other metadata 
sub-field are optional. The JData metadata record supports user-defined 
keywords.


### Data Presentation Keywords

#### Data Array Storage Keywords

In this document, we consider the following general data storage models:

-   a full array (empty, 1D, 2D, 3D or in higher dimensions)
-   a sparse array (empty, 1D, 2D, 3D or in higher dimensions)

`_ArraySize_, _ArrayType_, _ArrayData_,`
`_ArrayIsComplex_, _ArrayIsSparse_`

#### Meta-data Keywords

In this document, we defines the following JMesh keywords to represent 
non-geometric meta-data:

`_Author_, _CreationTime_, _Comment_`


Recommended File Specifiers
------------------------------

For the text-based JData file, the recommended file suffix is **".jdat"**; for 
the binary JData file, the recommended file suffix is **".ubjd"**.

The MIME type for the text-based JData document is 
**"application/jdata-text"**; that for the binary JData document is 
**"application/jdata-binary"**