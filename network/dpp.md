# NEATO `Direct Payload Protocol` Specification

Written by UsUsStudios

Revision 1 of August 27, 2026

---

This specification defines a connectionless transport-layer protocol comparable to the real-life User Datagram
Protocol, called the Direct Payload Protocol, or DPP. It is protocol without handshaking which in real networking
would be refered to as unreliable due to its lack of protection from data loss, but because NEETComputers does not
have any data loss, it does guarantee data integrity. It is comparable to the real-life User Datagram Protocol.

A DPP message should be sent as a table with the following entries:

```
{
    protocol: string = "dpp",
    target_port: integer,
    source_port: integer,
    payload: any
}
```

### "dpp":

The first argument must be a string consisting of exactly `dpp`, for the receiving program to confirm that this is
an DPP message. (Currently this is not useful, as DPP is the only transport-layer protocol, but in the future other
protocols may be created, some of which may not even be NEATO-defined.)

### target_port:

The second argument must be an integer that corresponds to the port number that this message is directed at. The
computer's operating system facility that handles networking events should only route the payload of the packet
to the program that is currently listening on this port.

### source_port

The third argument should either be an integer that corresponds to the port that the sending computer is listening
to for replies to this packet, or `-1` if the sending computer is not listening for replies. This information should
be passed down to the program that receives the packet, along with the payload, so that the program knows what
destination to target should it want to formulate a reply.

### payload

The sixth argument can be whatever the application-layer protocol requires. It can be a single value, or a table
(either an array or a dictionary table), as long as it is defined as such by the application-layer protocol. If
the payload is a dictionary table, then it SHOULD have a `protocol` entry with the protocol name/header for programs
to check against, and if the payload is an array table, then its first entry SHOULD be the protocol name/header, but
neither of these are required. It is against NEATO specification to allow a program to read, access or index the
payload if the target of the packet is not this computer and port.
