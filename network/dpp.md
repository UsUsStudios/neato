# NEATO `Direct Payload Protocol` Specification

Written by UsUsStudios

Revision 0 of August 26, 2026

---

This specification defines a transport-layer communication protocol, called Direct Payload Protocol, or DPP, as the
first layer for any application-layer protocol. It is connectionless protocol which in real networking would be
refered to as unreliable due to its lack of protection from data loss, but because NEETComputers does not have
any data loss, it does guarantee data integrity. It is comparable to the real-life UDP protocol.

A DPP message should be sent as the following arguments passed to a broadcasting function:

`("dpp", target_hostname, target_port, source_hostname, source_port, payload)`

### "dpp":

The first argument must be a string consisting of exactly `dpp`, for the receiving program to confirm that this is
an DPP message. (Currently this is not useful, as DPP is the only transport-layer protocol, but in the future other
protocols may be created, some of which may not even be NEATO-defined.)

### target_hostname:

The second argument must be a string consisting of the hostname of the target computer, or `localhost`. If it is
`localhost`, then the abstraction layer of the DPP protocol should add the message to the `Network` event queue of
the computer that sent the message as a broadcasting function would have, so that other programs on the computer can
receive the loopback message. If a message is received that has a `target_hostname` not equal to the hostname of this
computer, it is against NEATO specification to read or use the payload of the message, but of course, this cannot
be relied upon because malicious agents are still able to read or use for malicious purposes.

### target_port:

The third argument must be an integer that corresponds to the port number that this message should be routed to. If
this computer's operating system handles networking events itself, it should only route the message to the program
that has that port open. Otherwise, if programs handle networking events themselves, then they should re-queue messages
that have the correct `target_hostname`, but incorrect `target_port`, that they are looking for.

### source_hostname

The fourth argument should be a string consisting of the hostname of the computer that sent this message, EVEN IF
the `target_hostname` is `localhost`. This is so that computers that receive the `localhost` message can ensure that it
was not mistakenly broadcast to the global network, instead of staying only within the sending computer's own event queue.
Besides the use with `localhost`, `source_hostname` is used by programs to know which hostname to target should they
want to reply to message. The exception to this is if `target_hostname` is `localhost`, in which case the reply target
should be `localhost`.

Below is an example for how to handle these four arguments:

```lua
local hostname = "test_computer" -- the hostname of this computer
local listening_port = 12345
local e = event.getFirst("Network", "networkMessage")
local protocol, target_hostname, target_port, source_hostname = table.unpack(e) -- read the protocol, target_hostname and target_port

if protocol ~= "dpp" or (target_hostname ~= hostname and target_hostname ~= "localhost") then
    return -- completely disregard the message if the protocol or target_hostname do not match
end

if target_hostname == "localhost" then
    if source_hostname ~= hostname then
        return -- return because the computer that sent the loopback message is not this computer
    else
        source_hostname = "localhost" -- so that if you want to reply, you will do so through localhost
    end
end

if target_port ~= listening_port then
    event.queueEvent(e) -- disregard the message, but re-queue it, so that other programs can access it
    return
end

-- now you can finally use the DPP message you received!
```

### source_port

The fifth argument should either be an integer that corresponds to the port that the sending computer is listening
to for replies to this message, or `-1` if the sending computer is not listening for replies. This applies identically
even if `target_hostname` is `localhost`.

### payload

The sixth argument can be whatever the application-layer protocol requires. It can be a single value, or a table
(either an array or a dictionary table), as long as it is defined as such by the application-layer protocol. If
the payload is a dictionary table, then it SHOULD have a `protocol` entry with the protocol name/header for programs
to check against, and if the payload is an array table, then its first entry SHOULD be the protocol name/header, but
neither of these are required. It is against NEATO specification to read, access or index the payload if the target
of the message is not this computer and port.
