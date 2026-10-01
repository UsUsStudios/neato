# NEATO `Direct Payload Protocol` Specification

Written by UsUsStudios

Extension: `ext.dpp`

Version: 1

Requires: `core`

---

This specification defines a connectionless transport-layer protocol comparable to the real-life User Datagram
Protocol, called the Direct Payload Protocol, or DPP. It is a protocol without handshaking which in real networking
would be referred to as unreliable due to its lack of protection from data loss, but because NEET Computers does not
have any data loss, it does in practice guarantee data integrity.

An operating system that reports `ext.dpp` must provide the `dpp` API described at the end of this file, and must
handle DPP messages as described here.

A DPP message is sent as a table with the following entries:

```
{
    protocol: string = "dpp",
    target_port: integer,
    source_port: integer,
    payload: any
}
```

### protocol

The `protocol` field must be a string consisting of exactly `dpp`, for the receiving program to confirm that this is
a DPP message. (Currently this is not useful, as DPP is the only transport-layer protocol, but in the future other
protocols may be created, some of which may not even be NEATO-defined.)

### target_port

The `target_port` field must be an integer from 1 to 65535 that corresponds to the port number that this message is
directed at. The computer's operating system facility that handles networking events should only route the payload of
the packet to the program that is currently listening on this port, and must drop the message if there is none.

### source_port

The `source_port` field should either be an integer from 1 to 65535 that corresponds to the port that the sending
computer is listening to for replies to this packet, or `-1` if the sending computer is not listening for replies.
This information should be passed down to the program that receives the packet, along with the payload, so that the
program knows what destination to target should it want to formulate a reply.

### payload

The `payload` field can be whatever the application-layer protocol requires, as long as it can be sent over the
network. A payload is either a boolean, a number, a string, or a table whose keys and values are themselves such
values (tables may be nested, but must not refer to themselves). Functions, threads and userdata cannot be sent, and
metatables are not sent. The largest payload that can be sent is up to the operating system.

The payload can be a single value, or a table (either an array or a dictionary table), as long as it is defined as such
by the application-layer protocol. If the payload is a dictionary table, then it SHOULD have a `protocol` entry with
the protocol name/header for programs to check against, and if the payload is an array table, then its first entry
SHOULD be the protocol name/header, but neither of these are required.

### Delivery

DPP does not yet have any addressing of computers, because the link and internet layers have not been developed. Until
they are, a message reaches every computer that receives the underlying broadcast. Each operating system must deliver
it only to the program on that computer which is listening on `target_port`, and it is against NEATO specification to
allow a program to read, access or index the payload of a message that is not directed at a port it is listening on.
An operating system must silently drop received tables that are not valid DPP messages.

---

### The `dpp` API

This API is exposed to programs if the operating reports `ext.dpp`.

| Name         | Description                                                                                            | Arguments                                            | Returns                           |
| ------------ | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------- | --------------------------------- |
| dpp.listen   | Starts receiving messages for a port in this program. Only one program can listen on a port at a time. | port (int)                                           | true, or nil and an error message |
| dpp.unlisten | Stops receiving messages for a port. Ports are also released when the program ends.                    | port (int)                                           | nil                               |
| dpp.send     | Sends a message. `source_port` defaults to `-1`. Fails if the ports or the payload are not valid.      | target_port (int), payload (any), source_port (int?) | true, or nil and an error message |

TODO: change API to include NIP destination information (right?)

When a message arrives for a port that a program is listening on, that program receives the event

`dpp` (payload, source_port, target_port)

through [`event`](../api/event.md).

---

### Example usage

```lua
dpp.listen(80)
local _, payload, from = event.pull("dpp")
if from ~= -1 then
  dpp.send(from, {"reply", payload[2]}, 80)
end
```
