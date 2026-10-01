# NEATO

(Norms Expressed As a Tool for Operating (S)ystems)

NEATO is a standard that defines APIs and standard conventions for NEET Computers operating systems and their method
of booting.

The standards are not meant to be a large, strict ruleset, but rather a shim for general program compatibility over
the already existing functions that NEET Computers offers, and to make API changes in NEET Computers an issue for OS
maintainers to resolve, rather than the programs themselves. NEATO aims to include basic APIs which most applications
would need, such as for terminal emulation or standards for how program arguments should work.

An OS may comply with these body of standards however the maintainer should desire to. In order to maintain NEATO
compatibility, an operating system must provide NEATO software an environment which complies with NEATO specifications.

---

### Reasoning

NEET Computers encourages users to write their own operating systems, which means that application compatibility
may not be guaranteed from one operating system to the next. It also raises a problem of writing the code itself,
as developers may need to ship different versions of software for every operating system which the software maintainer
would like to deploy it on. NEATO aims to solve this by providing a body of standards for an operating system to
provide to a program, or, how the operating system itself may boot, for compatibility.

---

### Structure: core and extensions

NEATO is modeled on the way POSIX and RISC-V are organized. There is one small mandatory part, and everything else is
an optional, separately named extension. An operating system picks which extensions it implements and tells
programs which ones those are, so an OS never has to implement a large feature just to be compatible, and a program
never has to guess what it is running on.

There are three kinds of name:

| Name                | Meaning                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------- |
| `core`              | The mandatory base. Every NEATO environment provides it.                                    |
| `ext.<name>`        | A standard extension, defined by a specification in this repository. Example: `ext.screen`. |
| `x.<vendor>.<name>` | A non-standard extension defined by an OS or a third party. Example: `x.neetos.windows`.    |

Names are case sensitive and every segment consists only of letters and digits.

Querying. A program finds out what it is running on with the `sys` API (see [sys.md](api/sys.md)):

```lua
if sys.hasExtension("ext.screen", 1) then
  -- safe to use the screen API
end
```

`sys.getExtensions()` returns every supported extension together with its version. `core` is always present.

An extension that an OS does not implement must not appear to be half there. The OS must
report it as unsupported (`sys.hasExtension` returns `false`), and must not define any global, function or event name
that the extension specification defines. A program can therefore also test for a feature by checking for `nil`.

Every extension, `core` included, has one integer version, starting at 1. The version only changes for a
breaking change. Anything that an OS could reasonably decline to implement is never added to an existing extension, it
becomes a new extension instead. An OS implements exactly one version of each extension it reports, and a program
should check for the exact version it was written for. There is no version for NEATO as a whole.

An extension may require other extensions. An OS that reports an extension must also report
everything it requires.

Names in the `ext.` namespace listed below as reserved have no specification yet. An OS must not
report them as supported. This keeps the names free until a specification is written.

---

### Extension registry

| Name         | Version | Requires | Specification                                     | Contents                                                               |
| ------------ | ------- | -------- | ------------------------------------------------- | ---------------------------------------------------------------------- |
| `core`       | 1       |          | [api/](api/README.md), [common/](common/paths.md) | Lua environment, `sys`, `event`, `term`, `fs`, `print`, `CWD`, paths   |
| `ext.screen` | 1       | `core`   | [api/screen.md](api/screen.md)                    | NEET Computers `screen` API, possibly redirected to a window or layer. |
| `ext.dpp`    | 1       | `core`   | [network/dpp.md](network/dpp.md)                  | Direct Payload Protocol.                                               |

Reserved: `ext.headsup`, `ext.peripherals`, `ext.chip`, `ext.internet`, `ext.partitions`.

Bootloaders are not part of an operating system's program environment, so they are not extensions. They are covered
by their own specification in [boot/](boot/README.md), which carries its own version number.

---

NEATO program API definitions, like functions or events relating to the NEATO body of standards, are placed in the
`api` folder.

NEATO definitions about the NEATO compliant boot process are placed in the `boot` folder.

Common formats that are shared between NEATO program API definitions and the NEATO boot definitions are defined in
the `common` folder.

NEATO specifications for networking protocols are defined in the `network` folder.

Every specification file states, at its top, which extension it belongs to and that extension's version.
