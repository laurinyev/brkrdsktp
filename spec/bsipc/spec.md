# Broker Style Interprocess Communication
WARNING: THIS FIRST DRAFT ~~WAS WRITTEN BY AN IDIOT~~ IS A FIRST DRAFT.

**TODO: LAZY TO DO ERROR HANDLING TODAY**

## 1 Terminology
* platform protcol(abbreviated as `PP`): the means of connecting and interacting with the server
* broker: an implementation of this specification
* client: a process that is connected to the broker via a binding
* server: a client that advertizes a service to other clients
* protocol: a specification for how a server may act
* command: a one-time message sent from client to server requesting an action
* packet: a one-time message sent either direction with protocol-defined content
* length-type-data format: an entry formatted as `[length: 2 byte little endian unsigned integer][type: 1 byte integer][data, has the specified length]`

## 2 Data types
* `0x0`: 8-bit unsigned integer(length must be `1`)
* `0x1`: 8-bit signed integer(length must be `1`)
* `0x2`: 16-bit unsigned integer(length must be `2`)
* `0x3`: 16-bit signed integer(length must be `2`)
* `0x4`: 32-bit unsigned integer(length must be `4`)
* `0x5`: 32-bit signed integer(length must be `4`)
* `0x6`: 64-bit unsigned integer(length must be `8`)
* `0x7`: 64-bit signed integer(length must be `8`)
* `0x8`: UTF-8 encoded string(variable length)
* `0x9`: UTF-16 encoded string(variable length)
* `0xA`: UTF-32 encoded string(variable length)
* `0xB`: callable; content: ```[bound command ID: 2 byte little endian unsigned integer][number of arguments: 1 byte unsigned integer]{list of argument types: 1 byte each}[length of UTF-8 name: 1 byte unsigned integer]{UTF-8 name}```

## 3 Platform Protocols
For each target platform there must be a platform protocol that defines how commands and packets are sent and recieved.

**TODO: add requirements for binding writers so parity is guaranteed and bridging remains possible**

## 5 Data packets
The BSIPC protocol uses "packets" to transmit information between clients.
A packet must have the following strucutre:
```[protocol name: 4 letters in UTF-8][data length: 2 byte little endian uint][kind: 1 byte uint][packet data]```

The means of sending and accepting packets is PP-defined.

### 5.1 Packet kinds
Each protocol gets 256 packet kinds to work with. What all packet kinds mean is protocol-defined, except that `0` is reserved as a global for completion packets.

## 6 Sending comands
The means of sending commands is PP-defined.

Sending a command requires a command ID and arguments encoded as a sequence of length-type-data entries.

Sending a command returns a unique receipt number, and exactly one completion packet identified by the receipt number shall be sent upon completion.

Response values other than errors shall be returned via packets.

Commands shall be routed based on the protocol name and command ID, arguments may be checked against advertized signatures based on broker configuration.

### 6.1 Command identifiers
Command identifiers are `2` byte numbers, allowing a maximum of `UINT16_MAX` commands for each protocol. Command identifiers are scoped per-protocol.

### 6.2 Completion packets
A completion packet is a packet with packet kind `0` and the following format:
```[..header...][receipt number: 2 byte little endian unsigned integer][error code: 2 byte little endian unsigned integer]```
and optionally 
```[number of fields: 2 byte little endian unsigned integer]{protocol-defined length-type-data fields}```

If the packet size is,
* 0-3 bytes: malformed packet, this should be treated as an unrecoverable error
* 4 bytes: only an error code, no response
* 5+ bytes: error code + response data, responses are unchecked **for now**

### 6.3 Receipt numbers
Receipt numbers must be unique among currently outstanding commands.

## 7 Client-server metaprotocol
The client-server metaprotocol(4 letter code: `CSMP`) defines how a client may advertize and bind other protocols.

### 7.1 The protocol structure
A protocol is of the following structure:
```[number of fields: 4 byte little endian uint]{fields in a length-type-data format, the lengths are in bytes}```.

### 7.2 Listing available protocols
A client may list available protocols by sendign the `CSMP_LIST_AVAIL`(`0x0`) command.

The response packet will contain:
```[number of protocols found: 2 byte little endian uint]{four-letter protocol names}```

### 7.3 Binding a protocol
A client may bind a protocol via the `CSMP_BIND_PROTOCOL`(`0x1`) command with arguments:
```[protocol name: 4 letters in UTF-8]```

The response packet will contain the protocol structure of the protocol.

### 7.4 Advertizing a protocol
A client may advertize a protcol via the `CSMP_ADVERTIZE_PROTOCOL`(`0x2`) comand with arguments: 
```[protocol name: 4 letters in UTF-8][protocol structure(see 7.1)]```

Protocol names must be unique within a broker.

The advertising client becomes the server for that protocol and receives all commands addressed to it.

The means of accepting commands is PP-defined.
