# read_tree.md

# read_tree Function Documentation

---

## Overview

The `read_tree` function is a key utility within the **Tree and Commit Object Management** section of the pygit project. It plays a critical role in parsing Git tree objects, which represent directory structures within a Git repository. Specifically, a tree object contains entries for files and subdirectories, each entry holding metadata like file mode, filename, and the SHA-1 hash of the associated Git object (blob or sub-tree).

Understanding and using `read_tree` is essential for operations that inspect or manipulate the contents of a Git repository at the tree object level, such as listing directory contents, traversing repository snapshots, or building higher-level Git commands.

This documentation describes the purpose, usage, and internal workings of `read_tree`, along with examples and diagrams to clarify the structure of parsed tree entries.

---

## Function: `read_tree`

```python
def read_tree(sha1=None, data=None):
    """Read tree object with given SHA-1 (hex string) or data, and return list
    of (mode, path, sha1) tuples.
    """
    if sha1 is not None:
        obj_type, data = read_object(sha1)
        assert obj_type == 'tree'
    elif data is None:
        raise TypeError('must specify "sha1" or "data"')
    i = 0
    entries = []
    for _ in range(1000):
        end = data.find(b'\x00', i)
        if end == -1:
            break
        mode_str, path = data[i:end].decode().split()
        mode = int(mode_str, 8)
        digest = data[end + 1:end + 21]
        entries.append((mode, path, digest.hex()))
        i = end + 1 + 20
    return entries
```

---

### Purpose

- Parses a Git tree object from either:
  - A SHA-1 hash referencing the object stored in the Git object database, or
  - Raw `data` bytes containing the tree object content.

- Returns a list of tree entries, each represented as a tuple:
  `(mode, path, sha1)` where:
  
  - `mode`: File mode as an integer (e.g., `100644` for a normal file, `40000` for a directory).
  - `path`: Filename or directory name as a string.
  - `sha1`: SHA-1 hash of the referenced Git object (blob or subtree) as a hex string.

This function is foundational for interpreting the structure of a repository snapshot at a given tree object.

---

### Parameters

| Parameter | Type   | Description                                                    |
|-----------|--------|----------------------------------------------------------------|
| `sha1`    | str or None | The SHA-1 hash (hex string) identifying the tree object to read. Optional if `data` is provided. |
| `data`    | bytes or None | Raw bytes of a tree object. Optional if `sha1` is provided. |

**Note:** Exactly one of `sha1` or `data` must be provided.

---

### Return Value

- Returns a list of tuples, each being `(mode, path, sha1)`.

---

### Preconditions and Validations

- If `sha1` is provided, the function:
  - Uses `read_object` to fetch and decompress the object data.
  - Verifies the object type is `'tree'`.
- If neither `sha1` nor `data` is provided, raises a `TypeError`.
- Assumes the tree object data is well-formed according to Git's tree object format.

---

### How It Works (Step-by-Step)

1. **Source of Tree Data:**
   - If `sha1` is given:
     - Calls `read_object(sha1)` to retrieve the object type and raw data.
     - Asserts the object type is `'tree'`.
   - If `data` is provided directly, proceeds to decode it.

2. **Parsing Tree Entries:**
   - Initializes a cursor `i` at 0 to traverse the `data`.
   - In a loop (up to 1000 iterations to prevent infinite loops):
     - Finds the next null byte `\x00` starting from `i`. The null byte separates the mode/path string from the SHA-1 binary hash.
     - If no null byte is found (`end == -1`), parsing ends.
     - Decodes the bytes from `i` to `end` as a UTF-8 string, then splits on space to separate `mode_str` and `path`.
     - Converts `mode_str` from octal string to integer.
     - Extracts the next 20 bytes after the null byte as the SHA-1 digest (in binary).
     - Converts the digest to a hex string.
     - Appends `(mode, path, sha1)` tuple to the entries list.
     - Advances `i` pointer past the digest to continue parsing.

3. **Return:**
   - Returns the collected list of entries.

---

### Usage Example

```python
# Read and list entries in a tree object given its SHA-1 hash
tree_sha1 = '4b825dc642cb6eb9a060e54bf8d69288fbee4904'  # example SHA-1 for an empty tree
entries = read_tree(sha1=tree_sha1)

for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

**Sample output:**

```
Mode: 0o100644, Path: README.md, SHA-1: e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
Mode: 0o40000, Path: src, SHA-1: 3a1f39e1e1c3e5e7885a9f2e8b0e65f3e5f07a7a
```

---

### Explanation of the Git Tree Object Format

A Git tree object encodes multiple entries concatenated together. Each entry has the format:

```
<mode> <filename>\0<20-byte SHA-1 binary>
```

- `<mode>`: File mode in octal string (e.g., `"100644"` for a file).
- `<filename>`: File or directory name.
- `\0`: Null byte separator.
- `<20-byte SHA-1 binary>`: Raw binary SHA-1 hash of the object.

The `read_tree` function parses this raw byte structure into a list of easy-to-use tuples.

---

### ASCII Diagram of Tree Object Data Layout

```
+------------------------+----------------------+-----------------------+ ...
| "100644 README.md" 0x00 | 20-byte SHA-1 binary | "040000 src" 0x00      | ...
+------------------------+----------------------+-----------------------+ ...
```

Where:

- `"100644 README.md"` is the ASCII mode and filename string.
- `0x00` is the null terminator byte.
- `20-byte SHA-1 binary` is the raw SHA-1 hash bytes.
- The pattern repeats for each entry in the tree.

---

### Relation to Other Functions

- `read_tree` relies on `read_object` to fetch and decompress the tree object data from the Git object store when given a SHA-1.
- Used by functions like `cat_file` when pretty-printing tree objects.
- Supports higher-level Git operations that require walking or examining directory trees, such as `find_tree_objects`, `commit`, and `diff`.

---

### Summary

- `read_tree` effectively converts raw Git tree object data into a structured Python list.
- It abstracts the low-level binary format, enabling other components to work with directory trees in a natural way.
- Proper use requires understanding the Git object model and the distinction between blobs and trees.

---

# End of read_tree.md Documentation File Content