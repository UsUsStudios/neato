# NEATO `fs` API Specification

Extension: `core`

Version: 1

---

A NEATO compliant environment must provide a global `fs` API for working with files and directories. It replaces the
raw NEET Computers `files` API, whose administrative functions (creating, deleting and hiding partitions, choosing the
boot partition, and so on) are not part of `core`.

Every path taken by `fs` is in the format defined in [paths.md](../common/paths.md). It may be a full path
(`disk:partition:/dir/file`) or a path relative to [`CWD`](cwd.md). What a program may see or change is decided by the
operating system, which may refuse any operation.

On failure, functions that return a result return `nil` followed by an error message (a string). The message is for
humans and programs must not compare it. A path that is not valid according to `paths.md` raises an error instead.

| Name             | Description                                                                                                                                           | Arguments                    | Returns                                               |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- | ----------------------------------------------------- |
| fs.exists        | Returns whether a file or directory exists at the path.                                                                                               | path (string)                | boolean                                               |
| fs.isFile        | Returns whether the path is a file.                                                                                                                   | path (string)                | boolean                                               |
| fs.isDir         | Returns whether the path is a directory.                                                                                                              | path (string)                | boolean                                               |
| fs.list          | Returns the names of everything inside a directory, in no particular order. Names do not include a path.                                              | path (string)                | names (table of strings), or nil and an error message |
| fs.makeDir       | Creates a directory. The parent directory must already exist, and the path must not.                                                                  | path (string)                | true, or nil and an error message                     |
| fs.delete        | Deletes a file or an empty directory. A partition root cannot be deleted.                                                                             | path (string)                | true, or nil and an error message                     |
| fs.open          | Opens a file. See below.                                                                                                                              | path (string), mode (string) | handle (table), or nil and an error message           |
| fs.resolve       | Returns the full, normalized form of a path, without checking that it exists. See [paths.md](../common/paths.md). The disk is always given as its ID. | path (string)                | full path (string), or nil and an error message       |
| fs.getDisks      | Returns the disks that are present, in ascending order of slot.                                                                                       | none                         | disks (table of `{slot = int, id = string}`)          |
| fs.getPartitions | Returns the names of the partitions on a disk.                                                                                                        | disk (int or string)         | names (table of strings), or nil and an error message |

---

### Opening files

`mode` is one of:

| Mode | Meaning                                                                                 |
| ---- | --------------------------------------------------------------------------------------- |
| "r"  | Read. The file must exist.                                                              |
| "w"  | Write. The file is created if it does not exist, and emptied if it does.                |
| "a"  | Append. The file is created if it does not exist. All writes go to the end of the file. |

A trailing `b` (`"rb"`, `"wb"`, `"ab"`) is accepted and has no effect. Files are always binary, and no newline
translation happens.

A handle has the following methods, called with `:`. Calling any of them on a closed handle raises an error.

| Name         | Description                                                                                                                                                                                                                                                                  | Arguments                       | Returns                                                          |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- | ---------------------------------------------------------------- |
| handle:read  | Reads from the current position. `"l"` (the default) reads a line without its newline, `"a"` reads everything that is left (an empty string at the end of the file), and a number `n` reads up to `n` bytes. Returns `nil` at the end of the file for `"l"` and for numbers. | format (string or int?)         | data (string), or nil (end of file), or nil and an error message |
| handle:write | Writes strings or numbers at the current position.                                                                                                                                                                                                                           | ... (string or number)          | the handle, or nil and an error message                          |
| handle:seek  | Moves the position. `whence` is `"set"` (from the start), `"cur"` (the default) or `"end"`. Returns the new position, counted from the start.                                                                                                                                | whence (string?), offset (int?) | position (int), or nil and an error message                      |
| handle:flush | Makes sure everything written so far is stored.                                                                                                                                                                                                                              | none                            | true, or nil and an error message                                |
| handle:close | Closes the handle. Handles that are still open when the program ends are closed by the operating system.                                                                                                                                                                     | none                            | true                                                             |

Reading from a handle opened for writing, or writing to a handle opened with `"r"`, returns `nil` and an error message.

---

### Example usage

```lua
local f = assert(fs.open("notes.txt", "w"))
f:write("hello\n", 42, "\n")
f:close()

local f = assert(fs.open("notes.txt", "r"))
print(f:read("l"))   -> "hello"
print(f:read("a"))   -> "42\n"
f:close()
```

```lua
for _, d in ipairs(fs.getDisks()) do
  print(d.slot, d.id, table.concat(fs.getPartitions(d.id), ", "))
end
```
