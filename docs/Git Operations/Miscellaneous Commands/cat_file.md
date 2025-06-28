# cat_file.md

# Displaying Contents or Information About Git Objects

## Overview

The `cat_file.md` document provides detailed reference documentation for the `cat_file` function and its related utilities within the pygit project. This file is part of the "Object Handling" section in the overall pygit documentation tree, which focuses on operations related to Git objects, such as reading, finding, and displaying object contents.

The primary purpose of `cat_file` is to display the contents or metadata of Git objects given their SHA-1 hash prefixes, supporting multiple display modes including raw data output, size, type, and prettified views. This functionality emulates the behavior of the Git command `git cat-file`, enabling users and programs to inspect Git objects stored in the repository.

This file is especially significant because it bridges low-level object reading with user-facing output, aiding in debugging, verification, and understanding of Git object internals.

---

## Function Documentation

### `cat_file(mode, sha1_prefix)`

**Purpose:**  
Display the contents or information about a Git object identified by the SHA-1 hash prefix.

**Parameters:**  
- `mode` (str): The display mode. Valid values are:
  - `'commit'`, `'tree'`, `'blob'`: Output raw object data of the expected type.
  - `'size'`: Print the size (in bytes) of the object data.
  - `'type'`: Print the type of the object (e.g., `'commit'`, `'tree'`, `'blob'`).
  - `'pretty'`: Print a human-readable, formatted representation of the object.
- `sha1_prefix` (str): A prefix of the object's SHA-1 hash used to identify the object uniquely.

**Behavior:**  
1. Reads the object data and type corresponding to `sha1_prefix` using `read_object`.
2. Depending on `mode`:
   - For `'commit'`, `'tree'`, `'blob'` modes:
     - Confirms the object type matches `mode`.
     - Writes raw bytes of the object to standard output.
   - For `'size'`:
     - Prints the length of the object data.
   - For `'type'`:
     - Prints the object's type string.
   - For `'pretty'`:
     - If the object is a `'commit'` or `'blob'`, outputs raw data.
     - If the object is a `'tree'`, parses the tree entries and prints each entry's mode, type, SHA-1, and path in a formatted manner.
   - For any other `mode` value, raises a `ValueError`.

**Usage Example:**

```bash
# Display raw contents of a blob object
$ pygit cat-file blob 1a2b3c4d

# Print the size of a commit object
$ pygit cat-file size 3f4e5d6c

# Show the type of an object
$ pygit cat-file type a1b2c3d4

# Pretty-print a tree object
$ pygit cat-file pretty 9f8e7d6c
```

**Python Usage Example:**

```python
import pygit

# Print raw commit data to stdout
pygit.cat_file('commit', 'f3c2d1e4')

# Print size of object
pygit.cat_file('size', 'f3c2d1e4')

# Pretty-print tree object
pygit.cat_file('pretty', 'a1b2c3d4')
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Read and decompress a Git object from the object database using a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (str): Prefix of the SHA-1 hash identifying the object.

**Returns:**  
Tuple `(obj_type, data)` where:
- `obj_type` (str): The type of the object (e.g., `'commit'`, `'tree'`, `'blob'`).
- `data` (bytes): The raw decompressed content of the object.

**Behavior:**  
- Finds the full object file path corresponding to the prefix.
- Reads and decompresses the file data.
- Parses the object header to extract the object type and size.
- Verifies the size matches the data length.
- Returns the object type and data.

**Usage Example:**

```python
obj_type, data = read_object('f3c2d1e4')
print(f'Type: {obj_type}')
print(data.decode())
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parse a Git tree object and return its entries.

**Parameters:**  
- `sha1` (str, optional): SHA-1 hash of the tree object to read.
- `data` (bytes, optional): Raw tree data bytes. Required if `sha1` is None.

**Returns:**  
List of tuples `(mode, path, sha1)` where:
- `mode` (int): File mode of the entry.
- `path` (str): File or directory name.
- `sha1` (str): SHA-1 hash of the entry object.

**Behavior:**  
- If `sha1` is provided, reads the object data for that tree.
- Parses the tree data by reading entries until no more null-terminated entries exist.
- Extracts mode, path, and SHA-1 for each entry.

**Usage Example:**

```python
entries = read_tree(sha1='a1b2c3d4...')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

### `read_file(path)`

**Purpose:**  
Read the contents of a file as bytes.

**Parameters:**  
- `path` (str): Path to the file.

**Returns:**  
Bytes of the file content.

**Usage Example:**

```python
data = read_file('.git/objects/1a/2b3c4d...')
print(data)
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Locate the filesystem path of a Git object given a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (str): Prefix of the SHA-1 hash.

**Returns:**  
Path (str) to the object file in the `.git/objects` directory.

**Behavior:**  
- Requires at least two characters in prefix.
- Lists matching objects in the appropriate object subdirectory.
- Raises error if none or multiple matches.

**Usage Example:**

```python
path = find_object('1a2b3c')
print(f'Object path: {path}')
```

---

## ASCII Diagram: Object Storage Layout

```
.git/
└── objects/
    ├── 1a/
    │   └── 2b3c4d5e6f7g8h9i0jklmnopqrstuvwx (compressed object file)
    ├── ...
    └── zz/
        └── ...
```

- The first two characters of the SHA-1 form the directory name.
- The remaining 38 characters form the filename within that directory.
- Objects are stored compressed and contain a header `<type> <size>\0` followed by data.

---

## Summary

The `cat_file` functionality is a core utility in the pygit implementation to inspect Git objects. It leverages lower-level functions to locate, decompress, and parse objects, offering multiple viewing modes tailored for different use cases. This capability is essential for debugging repository state, understanding commit and tree contents, and verifying object integrity.

Users and developers can invoke `cat_file` directly to extract raw or formatted data from Git objects, aiding in deeper insights into the repository's data structure.