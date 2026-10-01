# NEATO BOOT

Boot related specifications. The main one you have to care about is the [bootloader.md](bootloader.md) file as it
defines how OS booting is configured via `boot.lua`.

The bootloader is not an extension. It is versioned on its own, at the top of its specification file, since it
never runs alongside the program environment described in [api](../api/README.md).
