# read_tree.md

# Reading Tree Objects and Parsing Their Entries

---

## Overview

The `read_tree.md` document provides a detailed explanation of reading Git tree objects and parsing their entries. Tree objects in Git represent directories and contain references to blobs (file contents) and other trees (subdirectories). This file explains how to access these objects from the Git object store, interpret their binary format, and extract meaningful information such as file modes, paths, and SHA-1 hashes of the contents. 

Situated within the **Object Management** section of the documentation tree, this document complements related files like `read_object.md` and `cat_file.md`. It is essential for understanding how Git organizes the directory structure internally and for implementing commands that inspect or manipulate trees, such as `git ls-tree` or `git cat-file`.

---

## Functions

### `read_tree(sha1=None, data=None)`

#### Purpose

Reads a Git tree object either by its SHA-1 hash or directly from raw data bytes, and parses its entries to return a list of tuples describing the contents. Each tuple contains the file mode, the path (filename or directory name), and the SHA-1 hash of the referenced object.

This function is fundamental to interpreting the tree objects that represent directory snapshots in Git.

#### Parameters

- `sha1` (str, optional): The SHA-1 hash (hex string) identifying the tree object to read. If provided, the function reads the object from the Git object store.
- `data` (bytes, optional): Raw data bytes of a tree object. If provided, the function parses the entries directly from this data.
  
At least one of `sha1` or `data` must be provided. If both are provided, `data` is ignored, and the object is read via `sha1`.

#### Preconditions

- If `sha1` is provided, it must correspond to a valid tree object in the Git repository.
- If `data` is provided, it must contain a valid Git tree object format.
- The function assumes that the data is properly decompressed and formatted as a Git tree object.

#### Operation Steps

1. **Read the Tree Object Data:**
   - If `sha1` is given, call `read_object(sha1)` to retrieve the object type and data bytes.
   - Assert that the object type is `'tree'`.
   - If `data` is given directly, use it as the tree object data.
   - If neither is provided, raise an error.

2. **Parse Entries:**
   - Initialize an index `i` to 0 and an empty list `entries`.
   - Loop to parse entries until no more null (`\x00`) byte is found:
     - Find the index of the next null byte from position `i`.
     - Extract the mode and path from the bytes before the null byte.
       - The mode is expressed as an octal string.
       - The path is the filename or directory name string.
     - The 20 bytes following the null byte represent the SHA-1 hash of the referenced object.
     - Append a tuple `(mode, path, sha1_hex)` to `entries`.
     - Update `i` to the position after the SHA-1 hash to parse the next entry.

3. **Return the List of Entries**

#### Returns

- List of tuples: Each tuple is `(mode: int, path: str, sha1: str)`.

#### Example Usage

```python
from pygit import read_tree

# Read tree by SHA-1 hash
tree_sha1 = '4b825dc642cb6eb9a060e54bf8d69288fbee4904'  # example SHA-1
entries = read_tree(sha1=tree_sha1)

for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

Output example:

```
Mode: 0o100644, Path: README.md, SHA-1: e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
Mode: 0o40000, Path: src, SHA-1: d670460b4b4aece5915caf5c68d12f560a9fe3e4
```

---

## Additional Context: Tree Object Format in Git

A Git tree object encodes directory entries in the following format:

```
<mode> <path>\0<20-byte SHA-1>
```

This repeats for each entry in the tree. The mode is an octal file mode (e.g., `100644` for normal files, `40000` for directories). The path is the filename or directory name. The SHA-1 is the binary hash of the object (blob or tree) referenced.

#### ASCII Diagram of Tree Object Structure

```
+-------------------------------------------------------+
| Entry 1:                                              |
|  <mode> <path> \0 <20-byte SHA-1>                     |
|                                                       |
| Entry 2:                                              |
|  <mode> <path> \0 <20-byte SHA-1>                     |
|                                                       |
| ...                                                   |
+-------------------------------------------------------+
```

Each entry is concatenated sequentially without explicit separators other than the null byte and the fixed 20-byte SHA-1 hash.

---

## Related Functions (for context)

- **`read_object(sha1_prefix)`**  
  Reads a raw Git object by SHA-1 prefix and returns a tuple `(object_type, data_bytes)`.
  
- **`find_object(sha1_prefix)`**  
  Finds the file path of the object in the Git object store for a given SHA-1 prefix.
  
- **`cat_file(mode, sha1_prefix)`**  
  Displays the contents or metadata of an object, supporting modes including `tree`, which internally uses `read_tree` to parse tree objects.

---

This document explains the core process of interpreting Git tree objects, enabling tools and commands to navigate the repository directory structure at the object level. Understanding `read_tree` is key to building functionalities such as displaying directory listings, inspecting commits, and managing Git objects programmatically.