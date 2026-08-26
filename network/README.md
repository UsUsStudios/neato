# NEATO API

NEATO specifications for networking protocols.

The NEATO Network module defines specifications for networking protocols between programs running on separate
computers or possibly between programs running on a single computer. These networking protocols are designed to be
used with the built-in `io.broadcastLocal(arguments)` function, as well as potential future wireless equivalents.
The module first defines a format that all protocols should follow for compatibility, and then defines protocols
for certain uses. In the future, a NEATO API module should be specified to add an abstraction layer for the basic
`io.broadcastLocal(arguments)` calls, and a higher abstraction layer for all the NEATO specified protocols.

### Protocol specifications:

- [Common protocol format(common.md)](common.md) - The format that all NEETComputers networking protocols should
  adhere to for compatibility purposes.
