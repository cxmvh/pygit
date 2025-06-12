# cat_file.md

# cat_file Function Documentation

---

## Overview

The `cat_file` function is a utility within the `pygit` project designed to display information or contents of Git objects based on a specified mode. It plays a crucial role in inspecting Git objects such as commits, trees, and blobs by either outputting their raw data, showing their size or type, or providing a human-friendly ("pretty") representation. This function is part of the "Object Management" section of the repository documentation, which focuses on reading, writing, and manipulating Git objects. Understanding `cat_file` is essential for users and developers looking to interact with Git internals or build tooling that requires insight into Git object contents.

---

## Function: `cat_file`

### Purpose

The `cat_file` function outputs information or contents of a specified Git object identified by its SHA-1 prefix. Depending on the mode parameter, it can:

- Output raw object data (`commit`, `tree`, `blob`).
- Print the size of the object (`size`).
- Print the object type (`type`).
- Print a prettified version of the object (`pretty`), which for trees lists their entries in a human-readable format.

### Parameters

- **mode** (`str`): Specifies the type of output to produce. Valid values are:
  - `'commit'` — Output raw commit object data.
  - `'tree'` — Output raw tree object data.
  - `'blob'` — Output raw blob object data.
  - `'size'` — Print the size in bytes of the object.
  - `'type'` — Print the object type (`commit`, `tree`, or `blob`).
  - `'pretty'` — Print a human-readable formatted version of the object.
  
- **sha1_prefix** (`str`): A SHA-1 hex string prefix identifying the Git object.

### Preconditions

- The SHA-1 prefix must uniquely identify an existing Git object in the repository.
- The mode must be one of the supported strings; otherwise, a `ValueError` is raised.
- When using `'commit'`, `'tree'`, or `'blob'` modes, the object found must match the specified type exactly.

### Operation Details

1. **Object Lookup and Reading**

   The function first calls `read_object(sha1_prefix)` to retrieve the object's full type and its decompressed data bytes.

2. **Mode Handling**

   - If the mode is `'commit'`, `'tree'`, or `'blob'`:
     - It verifies the object type matches the mode.
     - It writes the raw object data bytes directly to standard output (stdout) in binary mode.
   
   - If the mode is `'size'`:
     - It prints the size (length) of the object data.
   
   - If the mode is `'type'`:
     - It prints the object type as a string.
   
   - If the mode is `'pretty'`:
     - For `'commit'` and `'blob'` types, it writes the raw data to stdout.
     - For `'tree'` type:
       - It uses `read_tree` to parse the tree data into entries.
       - For each entry, it prints the mode (in octal), object type (`tree` or `blob`), SHA-1, and the file path.
     - For any other object type, it raises an assertion error as unhandled.
   
   - If the mode is none of the above, it raises a `ValueError`.

### Example Usage

```bash
# Display the raw contents of a blob object
python -c "import pygit; pygit.cat_file('blob', 'f572d396fae9206628714fb2ce00f72e94f2258f')"

# Print the size of a commit object
python -c "import pygit; pygit.cat_file('size', '1a410efbd13591db07496601ebc7a059dd55cfe9')"

# Show the type of an object
python -c "import pygit; pygit.cat_file('type', '4a202b346bb0fb0db7eff3cffeb3c70babbd2045')"

# Pretty-print a tree object
python -c "import pygit; pygit.cat_file('pretty', 'f4d5a7c6d8e6ae3402b3d7fb9e255390b0d6d72e')"
```

---

## Supporting Functions Summary

To fully understand `cat_file`, it's helpful to know about the key supporting functions it uses:

- **`read_object(sha1_prefix)`**: Locates and decompresses the Git object by SHA-1 prefix, returning its type and raw data bytes.

- **`read_tree(sha1=None, data=None)`**: Parses a tree object either by SHA-1 or raw data, returning a list of `(mode, path, sha1)` entries.

- **`find_object(sha1_prefix)`**: Finds the file path to the object in the Git object store based on the SHA-1 prefix.

- **`read_file(path)`**: Reads raw bytes from a file.

---

## ASCII Diagram: Object Type Handling in `cat_file`

```
+----------------+
|   cat_file()   |
+----------------+
        |
        v
+-------------------------+
| read_object(sha1_prefix)|
+-------------------------+
        |
        v
+-----------------------------+
| obj_type, data = (type, data)|
+-----------------------------+
        |
        +----------------------------+
        |                            |
        v                            v
+-----------+               +---------------------+
| mode in   |               | mode == 'size'       |
| ['commit',|               | -> print size(data)  |
| 'tree',   |               +---------------------+
| 'blob']   |
+-----------+
        |
        v
+---------------------------------------+
| Verify obj_type == mode                |
| Write raw data to stdout buffer       |
+---------------------------------------+
        |
        +--------------------------------+
        |                                |
        v                                v
+-----------------+             +---------------------+
| mode == 'type'  |             | mode == 'pretty'    |
| -> print obj_type|             +---------------------+
+-----------------+                    |
                                      v
                      +-------------------------------------------+
                      | if obj_type in ['commit', 'blob']:        |
                      |   write raw data to stdout                 |
                      | elif obj_type == 'tree':                    |
                      |   for each entry in read_tree(data):       |
                      |     print mode, type, sha1, path           |
                      | else:                                      |
                      |   raise AssertionError                      |
                      +-------------------------------------------+
```

---

## Detailed Code Listing of `cat_file`

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

## Summary

`cat_file` is a versatile function that bridges raw Git object data with user-friendly output, enabling inspection and verification of Git objects. Its integration with `read_object` and `read_tree` ensures accurate and efficient retrieval of object data. This function is foundational for debugging, scripting, or building Git-related tools that require low-level access to repository objects.