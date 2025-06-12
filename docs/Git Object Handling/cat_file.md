# cat_file.md

# Displaying Git Object Contents (`cat_file`)

## Overview

This document details the functionality for displaying Git object contents in various modes within the `pygit` repository. It explains how to access and present Git objects such as commits, trees, and blobs, along with auxiliary information like object size and type. The `cat_file` functionality provides multiple modes: raw content output, size display, type display, and a prettified view, enabling users and tools to inspect Git objects efficiently.

Situated within the **Git Object Handling** section of the documentation tree, `cat_file.md` complements the foundational object reading and encoding documents by focusing on output modes and presentation of Git object data. It integrates closely with key functions like `read_object` and `read_tree` and leverages object metadata to provide flexible and user-friendly views of repository data.

---

## Function Documentation

### 1. `cat_file(mode, sha1_prefix)`

**Purpose:**  
Outputs the contents or information about a Git object identified by the SHA-1 prefix in various modes. This function is analogous to the `git cat-file` command and is central to inspecting repository objects.

**Parameters:**  
- `mode` (str): Specifies the output mode. Valid values are:
  - `'commit'`, `'tree'`, `'blob'`: Output raw data bytes for the given object type.
  - `'size'`: Print the size (in bytes) of the object.
  - `'type'`: Print the type of the object (commit/tree/blob).
  - `'pretty'`: Print a prettified representation of the object.
- `sha1_prefix` (str): A prefix of the SHA-1 hash identifying the object.

**Operation:**  
1. Uses `read_object` to retrieve the object type and raw data bytes.
2. For `commit`, `tree`, or `blob` modes:
   - Validates the object type matches the mode.
   - Writes raw bytes of the object to standard output.
3. For `size` mode:
   - Prints the size of the object data in bytes.
4. For `type` mode:
   - Prints the object type string.
5. For `pretty` mode:
   - If the object is `commit` or `blob`, outputs raw data.
   - If `tree`, parses the tree contents using `read_tree` and prints formatted entries showing mode, type, SHA-1, and path.
   - Raises an error if the object type is unrecognized.
6. Raises an error if the mode is unrecognized.

**Example Usage:**

```python
# Display raw commit object data
cat_file('commit', 'a1b2c3d')

# Show the size of a blob object
cat_file('size', 'e4f5a6b')

# Print the type of a tree object
cat_file('type', 'deadbeef')

# Pretty print a tree object
cat_file('pretty', 'bada55')
```

---

### 2. `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object identified by a SHA-1 prefix from the object store and returns a tuple containing the object type and the raw data bytes.

**Parameters:**  
- `sha1_prefix` (str): The SHA-1 hash prefix identifying the object.

**Operation:**  
1. Uses `find_object` to locate the full object file path in the `.git/objects` directory.
2. Reads the compressed object file contents and decompresses it using `zlib`.
3. Splits the decompressed data into header and data:
   - Header format: `<object_type> <size>\0`
   - Extracts `obj_type` and `size`.
4. Verifies that the size matches the length of the data bytes.
5. Returns `(obj_type, data)`.

**Example Usage:**

```python
obj_type, data = read_object('a1b2c3d')
print(f"Object type: {obj_type}")
print(f"Data bytes: {data[:50]}...")  # print first 50 bytes
```

---

### 3. `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a Git tree object either by SHA-1 hash or raw data, returning a list of entries describing the contents of the tree.

**Parameters:**  
- `sha1` (str, optional): The SHA-1 hash of the tree object to read.
- `data` (bytes, optional): Raw tree object data; used if `sha1` is not provided.

**Operation:**  
1. If `sha1` is provided:
   - Calls `read_object` to fetch the tree data, asserting the object type is `'tree'`.
2. If `data` is provided, uses it directly; raises error if neither is given.
3. Parses the tree data format:
   - Entries consist of: `<mode> <path>\0<20-byte SHA-1>`.
4. Iterates through the data until no further entries are found.
5. Returns a list of tuples: `(mode, path, sha1_hex_string)`.

**Example Usage:**

```python
entries = read_tree(sha1='bada55')
for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

---

### 4. `find_object(sha1_prefix)`

**Purpose:**  
Locates the file path of the Git object matching the given SHA-1 prefix in the `.git/objects` directory.

**Parameters:**  
- `sha1_prefix` (str): SHA-1 hash prefix (at least 2 characters).

**Operation:**  
1. Validates prefix length (minimum 2 characters).
2. Looks under `.git/objects/<first two chars>/` for files starting with the remainder of the prefix.
3. If exactly one match is found, returns the full path.
4. Raises errors if no matches or multiple matches are found.

**Example Usage:**

```python
obj_path = find_object('a1b2c3')
print(f"Object path: {obj_path}")
```

---

### Supporting ASCII Diagram: Git Object Storage Layout

```
.git/
└── objects/
    ├── aa/
    │   ├── bbbbbb...  # Object file named by remaining SHA-1 chars
    │   └── ...
    ├── bb/
    │   └── cccccc...
    └── ...
```

- The first two characters of the SHA-1 form the directory name.
- The remaining 38 characters form the filename.
- Objects are stored compressed.

---

### 5. `read_file(path)`

**Purpose:**  
Reads raw bytes from a file at the specified path.

**Parameters:**  
- `path` (str): File path.

**Operation:**  
Opens the file in binary mode and reads its entire content.

**Example Usage:**

```python
data = read_file('.git/objects/aa/bbbbbb...')
print(f"Read {len(data)} bytes")
```

---

### Additional Notes

- `cat_file` depends heavily on the correctness of `read_object` and `read_tree` to interpret raw Git objects.
- Error handling ensures that the object type requested matches the actual object type stored.
- The `pretty` mode provides a human-readable format for tree objects, listing entries with modes, types, SHA-1 hashes, and paths, aiding inspection.
- The functions interact directly with the `.git` directory structure and raw object files, reflecting a low-level approach to Git internals.

---

# Summary

The `cat_file.md` document explains how to display Git object contents in multiple modes by reading the raw objects from the Git object store. It provides detailed insight into the `cat_file` function, which mimics `git cat-file` behavior, and supporting functions like `read_object`, `read_tree`, and `find_object` that underlie object retrieval and decoding. These functions together facilitate detailed inspection of Git repository internals, essential for Git tooling and debugging.