# cat_file.md

# Documentation for `cat_file.py`

---

## Overview

The `cat_file` module provides functionality to view the contents or metadata of Git objects identified by their SHA-1 prefixes. It supports multiple display modes including raw output of commit, tree, or blob objects, as well as displaying the object’s size, type, or a prettified version. This tool is essential for inspecting Git repository internals, aiding in debugging and understanding how Git stores and structures data.

Within the broader documentation tree, this file fits under the **Object Handling and Git Objects** section, complementing other modules that read, write, hash, and manage Git objects. It interfaces closely with functions that locate objects by SHA-1 prefix, decompress and parse object data, and interpret tree structures.

---

## Function Documentation

### Function: `cat_file(mode, sha1_prefix)`

**Purpose:**  
Outputs the contents or information about a Git object specified by the SHA-1 prefix, according to the given mode.

**Parameters:**  
- `mode` (str): Determines the output format. Valid values include:
  - `'commit'`, `'tree'`, `'blob'`: Output raw data bytes of the object, verifying type.
  - `'size'`: Print the size of the object in bytes.
  - `'type'`: Print the Git object type (`commit`, `tree`, `blob`).
  - `'pretty'`: Print a prettified version of the object. For commits and blobs, raw data is printed; for trees, a formatted listing of entries is shown.
- `sha1_prefix` (str): Prefix of the SHA-1 hash identifying the Git object.

**Operation:**
1. Calls `read_object(sha1_prefix)` to retrieve the object type and raw data.
2. Depending on `mode`:
   - For `'commit'`, `'tree'`, `'blob'`: checks the object type matches `mode`, then writes raw bytes to stdout.
   - For `'size'`: prints the length of the data.
   - For `'type'`: prints the object type string.
   - For `'pretty'`: 
     - If object is `commit` or `blob`, writes raw data to stdout.
     - If object is `tree`, parses entries using `read_tree()` and prints each entry with mode, type, SHA-1, and path in a human-readable format.
3. Raises a `ValueError` on unrecognized mode or type mismatch.

**Example Usage:**

```bash
# Show the raw contents of a blob object:
python -c "import pygit.cat_file as cf; cf.cat_file('blob', 'e69de29bb2d1d6434b8b29ae775ad8c2e48c5391')"

# Print the size of a tree object:
python -c "import pygit.cat_file as cf; cf.cat_file('size', '4b825dc642cb6eb9a060e54bf8d69288fbee4904')"

# Pretty print a tree object:
python -c "import pygit.cat_file as cf; cf.cat_file('pretty', '4b825dc642cb6eb9a060e54bf8d69288fbee4904')"
```

---

### Function: `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object from the object store by its SHA-1 prefix and returns its type and raw data.

**Parameters:**  
- `sha1_prefix` (str): SHA-1 prefix string identifying the object.

**Returns:**  
- Tuple `(obj_type, data)`:
  - `obj_type` (str): Git object type, e.g., `'commit'`, `'tree'`, `'blob'`.
  - `data` (bytes): Raw content bytes of the object.

**Operation:**
1. Calls `find_object(sha1_prefix)` to get the full path to the object file.
2. Reads the compressed object data from disk and decompresses it with zlib.
3. Parses the header to extract the object type and size.
4. Extracts the object data bytes after the null separator.
5. Validates the size matches the expected length.
6. Returns `(obj_type, data)`.

**Example Usage:**

```python
obj_type, data = read_object('e69de29bb2d1d6434b8b29ae775ad8c2e48c5391')
print(f"Type: {obj_type}")
print(f"Data length: {len(data)} bytes")
```

---

### Function: `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a Git tree object to extract its entries, returning a list of tuples describing each entry.

**Parameters:**  
- `sha1` (str, optional): SHA-1 hash of the tree object to read.
- `data` (bytes, optional): Raw tree data to parse (if `sha1` is not provided).

**Returns:**  
- List of tuples `(mode, path, sha1)` where:
  - `mode` (int): File mode bits (e.g., directory or blob).
  - `path` (str): File or directory name.
  - `sha1` (str): SHA-1 hash of the Git object.

**Operation:**
1. If `sha1` is provided, reads the tree object data using `read_object()` and asserts it is a tree.
2. Parses the raw tree data sequentially:
   - Reads mode and path until a null byte.
   - Reads the following 20 bytes as SHA-1 binary.
   - Converts SHA-1 binary to hex string.
   - Appends the tuple to the entries list.
3. Stops when no more entries are found or after a safe iteration limit.

**Example Usage:**

```python
entries = read_tree(sha1='4b825dc642cb6eb9a060e54bf8d69288fbee4904')
for mode, path, sha1 in entries:
    print(f"{oct(mode)} {path} {sha1}")
```

---

### Function: `find_object(sha1_prefix)`

**Purpose:**  
Locates the object file in the Git object store directory corresponding to a given SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (str): Prefix of the SHA-1 hash to locate.

**Returns:**  
- `path` (str): Filesystem path to the object file.

**Raises:**  
- `ValueError` if:
  - The prefix is less than 2 characters.
  - No objects found matching the prefix.
  - Multiple objects found matching the prefix.

**Operation:**
1. Validates prefix length.
2. Determines the object directory path based on the first two characters of the prefix.
3. Lists files in that directory matching the rest of the prefix.
4. Ensures exactly one match is found.
5. Returns the full path to the object file.

**Example Usage:**

```python
path = find_object('e69de29')
print(f"Object path: {path}")
```

---

### Function: `read_file(path)`

**Purpose:**  
Reads the contents of a file as bytes.

**Parameters:**  
- `path` (str): Filesystem path of the file to read.

**Returns:**  
- `bytes`: Contents of the file.

**Operation:**  
Opens the file in binary mode and reads all bytes.

**Example Usage:**

```python
data = read_file('.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391')
print(f"Read {len(data)} bytes")
```

---

## ASCII Diagram: Git Object Store Path Structure

```
.git/
└── objects/
    ├── e6/
    │   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
    ├── 4b/
    │   └── 825dc642cb6eb9a060e54bf8d69288fbee4904
    └── ...
```

- The first two characters of the SHA-1 hash form a directory.
- The remaining 38 characters form the file name within that directory.
- This structure facilitates efficient storage and lookup of Git objects.

---

## Summary

The `cat_file` functionality is a vital utility to examine Git objects for debugging, inspection, and learning purposes. By supporting multiple display modes and leveraging robust underlying functions to locate, decompress, and parse objects, it provides flexible and detailed access to Git’s internal data structures.

This documentation should enable users and developers to understand the usage and internal workings of the `cat_file` capability within the PyGit repository ecosystem.