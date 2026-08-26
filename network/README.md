# NEATO API

NEATO specifications for networking protocols.

The NEATO Network module defines specifications for networking protocols between programs running on separate
computers or possibly between programs running on a single computer. These networking protocols are designed to be
used with the built-in `io.broadcastLocal(arguments)` function, as well as potential future wireless equivalents.
The module first defines a format that all protocols should follow for compatibility, and then defines protocols
for certain uses. In the future, a NEATO API module should be specified to add an abstraction layer for the basic
`io.broadcastLocal(arguments)` calls, and a higher abstraction layer for all the NEATO specified protocols.

In all protocol specifications, the term "broadcasting function" refers to any `io.broadcastLocal(arguments)`-equivalent
function, including `broadcastLocal` itself. This includes any function that sends a series of arguments to all computers
in a certain network, including the global network.

### Protocol specifications:

- [NEET Datagram Protocol (ndp.md)](ndp.md) - The transport-layer protocol used as the first layer for any other
  application-level protocol.
