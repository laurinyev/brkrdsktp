# Broker Style Interprocess Communication
WARNING: THIS FIRST DRAFT ~~WAS WRITTEN BY AN IDIOT~~ IS WORK-IN-PROGRESS.

After connecting through a binder, a process can either advertize a protocol or use one.

# The protocol structure
The protocol structure contains a four-letter protocol name, as well as a list of fields.

# Listing available protocols
A client may list available protocols by sendign the `BSIPC_LIST_AVAIL` command.

The response packet will contain a list of all supported protocols terminated with a zero.

# Querying a protocol
A client may query a protocol via the `BSIPC_QUERY_PROTOCOL` function.

The respinse packet will contain the protocol structure of the protocol.

# Advertizing a protocol
To be added.
