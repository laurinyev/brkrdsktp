# Broker Style Interprocess Communication
WARNING: THIS FIRST DRAFT ~~WAS WRITTEN BY AN IDIOT~~ IS WORK-IN-PROGRESS.

## Data packets
The BSIPC protocol uses "packets" to transmit information between clients and the server.
A packet must have the following strucutre:
```[protocol name: 4 letters in UTF-8][packet lentght: 2 byte little endian uint][kind: 1 byte uint][packet data]```

The means of sending packets is binding-defined.

## Sending comands
The means of sending commands is binding-defined.

Sending commands must take the command ID and command-defined arguments and return a result through binding-defined means.

## Client-server metaprotocol
The client-server metaprotocol(4 letter code: `CSMP`) defines how a client may advertize and bind other protocols.

### The protocol structure
A protocol is of the following structure:
```[number of fields: 4 byte little endian uint]{fields in a length_type_data format, the lengths are in bytes}```.

There are 4 field types:
* `BSIPC_FIELD_CONSTANT_INT`(0x0): a constant signed integer, must be 1,2,4 or 8 bytes long.
* `BSIPC_FIELD_CONSTANT_UINT`(0x1): a constant unsigned integer, must be 1,2,4 or 8 bytes long.
* `BSIPC_FIELD_STRING_UTF8`(0x2): a constant UTF-8/ASCII string
* `BSIPC_FIELD_CALLABLE`(0x3): a field that can be called with certain arguments

The callable field has the folowing content:
```
[bound command ID: 4 byte little endian unsigned integer][number of arguments: 1 byte unsigned integer]
{list of argument types: 1 byte each}[length of UTF-8 name: 1 byte unsigned integer]{UTF-8 name}```

Argument types follow the same types as arbitrary data packet types.

### Listing available protocols
A client may list available protocols by sendign the `BSIPC_LIST_AVAIL`(0x0) command.

The response packet will contain:
```[number of protocols found: 2 byte little endian uint]{four-letter protocol names}```

### Binding a protocol
A client may bind a protocol via the `BSIPC_BIND_PROTOCOL`(0x1) command with arguments:
```[protocol name: 4 letters in UTF-8]```

The response packet will contain the protocol structure of the protocol.

### Advertizing a protocol
**TO BE ADDED**
