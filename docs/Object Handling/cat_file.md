# cat_file.md

# pygit.cat_file — Display Git Object Contents

## Overview

The `cat_file` function is a utility in the `pygit` project that enables inspection and display of Git object contents stored in a repository. It supports multiple modes for viewing Git objects by their SHA-1 prefix hash, including raw data output (`commit`, `tree`, or `blob`), retrieving the object size, type, or a prettified human-readable view.

This file is part of the **Object Handling** section in the overall pygit documentation tree, which groups functions related to reading, locating, and displaying Git objects. The ability to display object contents is foundational for debugging, exploring repository state, and implementing higher-level Git commands.

---

## Function: `cat_file(mode, sha1_prefix)`

### Purpose

Display the contents or metadata of a Git object referenced by the given SHA-1 prefix. Depending on the `mode` argument, the function can output raw object bytes, size, type, or a pretty-printed representation.

### Parameters

- `mode` (`str`): Specifies the output mode. Valid values are:
  - `'commit'` — output raw commit object data.
  - `'tree'` — output raw tree object data.
  - `'blob'` — output raw blob object data.
  - `'size'` — print the size (in bytes) of the object data.
  - `'type'` — print the type of the object (`commit`, `tree`, `blob`).
  - `'pretty'` — print a human-readable formatted view of the object.
- `sha1_prefix` (`str`): The prefix of the SHA-1 hash identifying the Git object.

### Behavior

1. Read the full object from the repository using the SHA-1 prefix.
2. Validate the object type if a specific raw output mode (`commit`, `tree`, or `blob`) is requested.
3. Depending on the mode:
   - For raw output modes, write the object data bytes directly to stdout.
   - For `'size'`, print the length of the object data.
   - For `'type'`, print the object type string.
   - For `'pretty'`:
     - If the object is a `commit` or `blob`, print the raw data.
     - If the object is a `tree`, parse and display each tree entry in a formatted list.
4. Raise a `ValueError` for invalid modes or type mismatches.

### Raises

- `ValueError` if the object type does not match the requested mode (for raw modes).
- `ValueError` if an unexpected `mode` string is provided.

---

### Code Implementation

```python
def cat_file(mode, sha1_prefix):
    """Write the contents of (or info about) object with given SHA-1 prefix to
    stdout. If mode is 'commit', 'tree', or 'blob', print raw data bytes of
    object. If mode is 'size', print the size of the object. If mode is
    'type', print the type of the object. If mode is 'pretty', print a
    prettified version of the object.
    """
    obj_type, data = read_object(sha1_prefix)
    if mode in ['commit', 'tree', 'blob']:
        if obj_type != mode:
            raise ValueError('expected object type {}, got {}'.format(
                    mode, obj_type))
        sys.stdout.buffer.write(data)
    elif mode == 'size':
        print(len(data))
    elif mode == 'type':
        print(obj_type)
    elif mode == 'pretty':
        if obj_type in ['commit', 'blob']:
            sys.stdout.buffer.write(data)
        elif obj_type == 'tree':
            for mode, path, sha1 in read_tree(data=data):
                type_str = 'tree' if stat.S_ISDIR(mode) else 'blob'
                print('{:06o} {} {}\t{}'.format(mode, type_str, sha1, path))
        else:
            assert False, 'unhandled object type {!r}'.format(obj_type)
    else:
        raise ValueError('unexpected mode {!r}'.format(mode))
```

---

### Usage Examples

#### Example 1: Display Raw Commit Object

```bash
$ pygit cat-file commit 1a2b3c4d
```

This command prints the raw commit object contents identified by the SHA-1 prefix `1a2b3c4d`.

#### Example 2: Show Object Size

```bash
$ pygit cat-file size 1a2b3c4d
```

Outputs the size in bytes of the object with SHA-1 prefix `1a2b3c4d`.

#### Example 3: Show Object Type

```bash
$ pygit cat-file type 1a2b3c4d
```

Prints the object type — for example, `commit`, `tree`, or `blob`.

#### Example 4: Pretty-Print a Tree Object

```bash
$ pygit cat-file pretty 1a2b3c4d
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    README.md
040000 tree 4b825dc642cb6eb9a060e54bf8d69288fbee4904    src
```

This lists the tree entries with mode, type, SHA-1, and path in a readable format.

---

## Supporting Functions Overview

The `cat_file` function relies on these key helper functions:

- **`read_object(sha1_prefix)`**: Reads and decompresses the Git object, returning a tuple `(object_type, data_bytes)`.
- **`read_tree(sha1=None, data=None)`**: Parses a tree object's data, returning a list of `(mode, path, sha1)` tuples.
- **`find_object(sha1_prefix)`**: Finds the file path of the object in the `.git/objects` directory matching the given SHA-1 prefix.
- **`read_file(path)`**: Reads raw bytes from a filesystem path.

---

## ASCII Diagram: Object Type Flow in `cat_file`

```
+--------------------------+
| cat_file(mode, sha1)     |
+--------------------------+
          |
          v
+--------------------------+
| read_object(sha1)        |
| -> (obj_type, data)      |
+--------------------------+
          |
          +----------------------------+
          |                            |
          v                            v
Raw modes: 'commit', 'tree', 'blob'   Other modes
          |                            |
          |                            |
Mode matches obj_type?          mode == 'size' -> print len(data)
          |                            |
          |                            |
   Write raw data               mode == 'type' -> print obj_type
          |                            |
          |                            |
          +----------------------------+
          |
          v
mode == 'pretty'?
          |
          +----------------------------+
          |                            |
          v                            v
obj_type in ['commit', 'blob']    obj_type == 'tree'
          |                            |
Write raw data                read_tree(data) -> list entries
                              For each entry:
                              print formatted line
```

---

## See Also

- [`read_object`](read_object.md): Reading and parsing Git objects.
- [`read_tree`](read_tree.md): Parsing Git tree objects.
- [`find_object`](find_object.md): Finding objects in the Git object database.
- [`cat_file` command](https://git-scm.com/docs/git-cat-file) (Git CLI reference).

---

# End of cat_file.md