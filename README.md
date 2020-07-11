path
====

> File system path abstractions for ZZ

## Installation

Add this to your `zz.toml` file:

```toml
[dependencies]
path = "*"

[repos]
path = "git://github.com/zzmodules/path"
```

## Usage

```c++
using path
```

## API

### `Path`

An immutable structure that wraps a `char *` that is treated as a file path.

#### `new p = path::from(pathspec)`

`Path` struct constructor from a given `char *` pathspec.

```c++
using path

new p = path::from(".");
```

#### `new p = path::parent(child_path)`

`ParentPath` (`Path`) struct constructor from a given child `Path` pointer.

```c++
using path

new p = path::from("/home/user");
new parent = path::parent(&p);
```

#### `p.length()`

Returns the length in bytes for the `Path` string.

```c++
using log
using path

new p = path::from(".");
log::info("%lu", p.length()); // 1
```

#### `p.cstr()`

Returns a `char *` string of this `Path`.

```c++
using log
using path

new p = path::from("./zz.toml");
log::info("%s", p.cstr()); // "./zz.toml"
```

#### `p.ancestors()`

Returns an `Ancestors` struct that implements an _iterator_ interface
that will traverse all ancestors in this `Path`, including itself.

```c++
using log
using path

new p = path::from("/home/user/projects/project/zz.toml");
let mut ancestors = p.ancestors();

while !ancestors.ended {
  let ancestor = ancestors.next();
  log::info("%s", ancestor.cstr());
}
```

#### `p.extension()`

Returns a `char *` that represents the "extension" part of a `Path` string.
This function will return an empty string if an extension cannot be found.

```c++
using log
using path

new p = path::from("zz.toml");
log::info("%s", p.extension()); // ".toml"
```

#### `p.is_absolute()`

Returns `true` if the `Path` is an _absolute_ path, otherwise `false`.

```c++
using log
using path

new p = path::from("/files/zz.toml");
if p.is_absolute() {
  log::info("%s is an absolute path", p.cstr());
}
```

#### `p.is_relative()`

Returns `true` if the `Path` is a _relative_ path, the opposite of an
_absolute_ path, otherwise `false`.

```c++
using log
using path

new p = path::from("./zz.toml");
if p.is_relative() {
  log::info("%s is a relative path", p.cstr());
}
```

#### `p.is_root()`

Returns `true` if the path is the root path.

```c++
using log
using path

new p = path::from("/"); // or "c:\\"
if p.is_root() {
  log::info("%s is the root path", p.cstr());
}
```

#### `p.has_root()`

Returns `true` if the path contains the root path in it.

```c++
using log
using path

new p = path::from("/users"); // or "c:\\Users"
if p.has_root() {
  log::info("%s has the root path", p.cstr());
}
```

#### `p.to_path_buffer()`

Converts this `Path` to a `PathBufferBox` , a container for a `PathBuffer+`
with a fixed size of `path::MAX_PATH_BUFFER_BYTES`.

```c++
using log
using path

new p = path::from("/home");
let buffer = p.to_path_buffer();
buffer.push("users");
log::info("%s", buffer.cstr()); // "/home/users"
```

#### `p.display()`

Implements `display()` to return a safe, nullterm `char *`.

```c++
using log
using path

new p = path::from("./zz.toml");
log::info("%s", p.display()); // "./zz.toml"
```

### `PathBuffer+`

A mutable tail sized structure for path building. The maximum tail size of
this struct can ba `path::MAX_PATH_BUFFER_BYTES`.

#### `new+T b = path::buffer(pathspec)`

`PathBuffer+` constructor from an initial `char *` pathspec.

```c++
using path

new+1024 b = path::buffer(".");
```

#### `new b = path::canonicalize(pathspec, error)`

`PathBufferBox` (`PathBuffer+`) constructor from an initial `char *` pathspec
that is * canonicalized. An `err::Err+ mut` error must be given to catch
system errors that may occur during normalization (`getcwd()`, `realpath()`)
or canonicalization (`realpath()`).

```c++
using err
using path

new+1024 mut error = err::make();
new b = path::canonicalize("../path/to/resolve", &error);

if error.check() {
  // canonicalization failed, check error trace
}
```

#### `b.string`

A tail sized `String+` container for building the underlying path
string.

#### `b.path`

A `Path` with a pointer to the underlying memory in the `PathBuffer+`
string container.

#### `b.normalize(error)`

Normalizes the current pathspec for a `PathBuffer+`. An `err::Err+ mut`
error must be given to catch system * errors that may occur during
normalization (`getcwd()`, `realpath()`).

```c++
using err
using path

new+1024 mut error = err::make();
new+1024 b = path::buffer("/home/../path/dir");

b.normalize(&error);

if error.check() {
  // normalization failed, check error trace
}
```

#### `b.length()`

Returns the length in bytes for the `PathBuffer+` string.

#### `b.cstr()`

Returns a `char *` string of this `PathBuffer+`.

#### `b.as_path()`

Returns an immutable `Path` that represents this `PathBuffer+`

```c++
using path


new+1024 b = path::buffer(".");
let p = b.as_path();
```

#### `b.as_path_ptr()`

Returns a immutable `Path *` that represents a pointer to this `PathBuffer+`
as a `Path` type.

```c++
using path

new+1024 b = path::buffer(".");
let p = b.as_path_ptr();
```

#### `b.push(pathspec)`

Push a `char *` pathspec component to the buffer returning `true`
upon success, otherwise `false`.

```c++
using log
using path

new+1024 b = path::path_buffer();
b.push("path");
b.push("to");
b.push("file");
log::info("%s", b.display()); // "/path/to/file"
```

#### `b.pop()`

Pops a pathspec component from the buffer returning `true` upon success,
otherwise `false`.

```c++
using log
using path

new+1024 b = path::path_buffer();
b.push("path");
b.push("to");
b.push("file");
b.pop();
log::info("%s", b.display()); // "/path/to"
```

#### `b.append(path_buffer)`

Appends a `PathBuffer+` components to this `PathBuffer+` returning
`true` upon success, otherwise `false`.

```c++
using log
using path

new+1024 b = path::path_buffer();
new+1024 joined = path::buffer("/home");
b.push("path");
b.push("to");
b.push("file");

log::info("%s", b.display()); // "/path/to/file"
joined.append(&b);

log::info("%s", joined.display()); // "/home/path/to/file"
```

#### `b.clear()`

Clears the buffer.

```c++
using log
using path

new+1024 b = path::path_buffer();
b.push("path");
b.push("to");
b.push("file");
b.clear();
log::info("%s", b.display()); // ""
```

#### `b.ancestors()`

Returns an `Ancestors` struct that implements an _iterator_ interface
that will traverse all ancestors in this `PathBuffer+`, at this moment
in time, including itself.

#### `b.extension()`

Returns a `char *` that represents the "extension" part of the
`PathBuffer+` string. This function will return an empty string if an
extension cannot be found.

#### `b.is_absolute()`

Returns `true` if the `PathBuffer+` is an _absolute_ path, otherwise `false`.

#### `b.is_relative()`

Returns `true` if the `PathBuffer+` is a _relative_ path, the opposite of an
_absolute_ path, otherwise `false`.

#### `b.is_root()`

Returns `true` if the `PathBuffer+` is the root path.

#### `b.has_root()`

Returns `true` if the `PathBuffer+` contains the root path in it.

#### `b.display()`

Implements `display()` to return a safe, nullterm `char *`.

### `Ancestors`

A structure that implements an _iterator_ interface to represent the
eventual ancestors of a path.

```c++
let mut ancestors = p.ancestors();

while !ancestors.ended {
  let ancestor = ancestors.next();
  // `ancestor` is a `Path` type
}
```

#### `a.ended`

A boolean to indicate if the `Ancestors` iterator has reached the end of
the path tree.

#### `a.next()`

Return a `Path` struct with a pointer to the next ancestors path.

#### `a.display()`

Implements `display()` to return a safe, nullterm `char *`.

### `path::separator()`

Get the operating system specific path separator as a `char *`. On
Windows, this will be `"\\"`, otherwise `"/"`.

#### `path::separator::character()`

Get the operating system specific path separator character., On Windows,
this will be `'\\'`, otherwise `'/'`.

### `path::delimiter()`

Get the operating system specific `PATH` delimiter as a `char *`. On
Windows, this will be `";"`, otherwise `":"`.

#### `path::delimiter::character()`

Get the operating system specific `PATH` delimiter character. On
Windows, this will be `';'`, otherwise `':'`.

## License

MIT
