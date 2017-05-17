# protograph

![PROTOGRAPH](https://github.com/bmeg/protograph/blob/master/resources/public/connections.jpg)

# what is protograph?

Protograph is a protocol for describing the transformation from any given schema into a set of graph vertexes and edges. 

Input for Protograph is a stream of messages described in a [Protocol Buffers schema](https://developers.google.com/protocol-buffers/), and a `protograph.yml` file that describes how to transform these messages into vertexes and edges.

To a small degree the `protograph.yml` description concerns serialization of the actual values stored in the vertex or edge, but largely Protograph is concerned with how to *link* the separate entities together. In order to create a graph out of many separate messages, the values in each message that describe their identity and links to other entities must be consistent with one another. Largely this consistency is generated before the messages arrive to be processed by Protograph, but Protograph maintains this consistency and in some cases generates additional references during its processing.

In general when referring to *links* this can mean a vertex connecting through an edge to another vertex, but it could also mean an edge connecting to either terminal vertex or even a partial edge referencing its other half.

## protograph describes a property graph

To Protograph, vertexes and edges are able to contain properties: ie, key/value pairs which are associated to a given vertex or edge. These properties are arbitrary structures containing one of these types:

* string
* number (integers or doubles)
* list of any mixed values
* map of strings to any value (string, number, list or map)

While vertexes and edges can both have edges, they differ in that a vertex just has a gid and label, whereas an edge contains in addition a gid for both `from` and `to` terminals, as well as the vertex label of each terminal.

    # input to Protograph representing a single Variant
    {"sample": "biosample:CCLE:1321N1_CENTRAL_NERVOUS_SYSTEM",
     "referenceName": "1",
     "start": "10521380",
     "end": "10521380",
     "referenceBases": "A",
     "alternateBases": ["-"]}

    # output from Protograph
    {"label": "Variant"
     "gid": "variant:1:10521380:10521380:A:-"
     "properties": {
       "referenceName": "1",
       "start": "10521380",
       "end": "10521380",
       "referenceBases": "A",
       "alternateBases": ["-"]}}}

    {"label": "variantInBiosample",
     "fromLabel": "Variant",
     "from": "variant:1:10521380:10521380:A:-"
     "toLabel": "Biosample",
     "to": "biosample:CCLE:1321N1_CENTRAL_NERVOUS_SYSTEM",
     "gid": "(variant:1:10521380:10521380:A:-)->(biosample:CCLE:1321N1_CENTRAL_NERVOUS_SYSTEM)",
     "properties": {}}

## protograph works with typed messages

Protograph directives are partitioned by type. When creating a protobuffer schema you declare a series of message types, and in `protograph.yml` you refer to these type names when declaring how each message will be processed.

    # a typed message
    - label: Variant

## each message has a role

When handling messages as they come in, some of the messages translate directly to vertexes, others are edges, while even others are "partial edges": representing one terminal or another (incoming or outgoing). The role of a message is distinct from the label of a message: the label represents the input type, whereas the role reflects its place in the output.

Currently there are three roles, `Vertex`, `Edge` and `Partial Edge`. In the future more may appear, but for now this is sufficient.

    # this message will become a vertex
    role: Vertex

## each message type has a gid

Gids are one of the key concepts of Protograph. A gid (global identifier) refers to an identifier that can be entirely constructed *from the message itself*. Each message type declares a gid template that accepts the message as an argument and constructs the gid from values found within.

    # this gid is composed of several properties
    gid: "variant:{{referenceName}}:{{start}}:{{end}}:{{referenceBases}}:{{alternateBases}}"

## messages reference one another through gids

Gids are used to link messages together. Typically a message will contain a gid for another message under some property in a string (for a single link) or list (for a multitude of links). Sometimes these references will be embedded inside an inner map, or list of maps. Protograph enables you to specify references anywhere they may live.

    # this variant came from a sample
    "sample": "biosample:CCLE:1321N1_CENTRAL_NERVOUS_SYSTEM"

## transformations are comprised of directives

In general, you specify a transformation for a given message type by describing what to do for each key in the message. Each key can contain any arbitrary data, so directives must also handle a variety of message types. Describing what the directives are, what they do, and how to invoke them is the bulk of understanding how to use Protograph.

    # how to transform a variant
    actions:
      - field: alternateBases
        join_list:
          delimiter: ","
  
# how to write protograph

Protograph has a protocol buffer schema defined [here](https://github.com/bmeg/protograph/blob/master/src/main/proto/protograph/schema/Protograph.proto).



