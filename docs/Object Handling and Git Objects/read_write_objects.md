# read_write_objects.md

## Overview

This document provides detailed information on reading and writing Git objects within the repository. It covers essential operations including reading objects by their SHA-1 hash or prefix, writing objects back to the Git object store, parsing tree objects that represent directory structures, and locating objects based on abbreviated SHA-1 prefixes. This file plays a critical role in the "Object Handling and Git Objects" section of the documentation tree, complementing related files such as `object_handling.md` and `object_storage.md`. Mastery of these functions is foundational for understanding how Git internally manages data blobs, trees, commits, and facilitates object retrieval and storage.

---

## Function Documentation

### `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object identified by a SHA-1 prefix from the object store and returns a tuple containing the object type (e.g., 'commit', 'tree', 'blob') and the raw data bytes of the object.

**Parameters:**  
- `sha1_prefix` (`str`): A prefix string representing the first part of the object's SHA-1 hash. Must uniquely identify a single object.

**Operation Steps:**  
1. Calls `find_object(sha1_prefix)` to locate the file path of the object in the `.git/objects` directory.  
2. Reads the compressed object file and decompresses it using zlib.  
3. Parses the object header (format: `<type> <size>\0`) to extract the object type and expected data size.  
4. Extracts the object data after the header and verifies that the data size matches the expected size.  
5. Returns a tuple `(obj_type, data)`.

**Example Usage:**

```python
obj_type, data = read_object('a1b2c3d4')
print(f'Object type: {obj_type}')
print(f'Data bytes length: {len(data)}')
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Finds the file path of a Git object in the object store given a SHA-1 prefix. Ensures the prefix uniquely identifies an object.

**Parameters:**  
- `sha1_prefix` (`str`): The first 2 or more characters of the SHA-1 hash.

**Operation Steps:**  
1. Validates that the prefix length is at least 2 characters.  
2. Uses the first two characters of the prefix to locate the subdirectory within `.git/objects/`.  
3. Lists files in the directory that start with the remainder of the prefix.  
4. Raises an error if zero or multiple matching objects are found.  
5. Returns the full path to the uniquely identified object file.

**Example Usage:**

```python
obj_path = find_object('a1b2')
print(f'Object stored at: {obj_path}')
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a Git tree object and returns a list of its entries, each representing a file or subtree with mode, path, and SHA-1.

**Parameters:**  
- `sha1` (`str`, optional): SHA-1 hash of the tree object to load and parse.  
- `data` (`bytes`, optional): Raw data of a tree object to parse directly.  
*Exactly one of `sha1` or `data` must be provided.*

**Operation Steps:**  
1. If `sha1` is given, uses `read_object` to fetch the tree data and confirms the object type is 'tree'.  
2. If `data` is provided directly, uses it as the tree contents.  
3. Iteratively parses each tree entry:  
   - Extracts the mode (as an octal number) and path separated by a space, terminated by a null byte.  
   - Reads the subsequent 20 bytes as the SHA-1 binary digest of the entry.  
4. Converts the SHA-1 digest to hexadecimal string and appends the tuple `(mode, path, sha1)` to the entries list.  
5. Returns the list of entries.

**Example Usage:**

```python
entries = read_tree(sha1='4b825dc642cb6eb9a060e54bf8d69288fbee4904')
for mode, path, sha1 in entries:
    print(f'{oct(mode)} {path} {sha1}')
```

**ASCII Diagram:**

```
Tree Object Data Structure:

+----------------+----------------+-------------------+--------+
| Mode (octal)   | Path (string)  | Null byte (0x00)  | SHA-1  |
| (e.g. 100644)  | (e.g. "file")  |                   | (20 b) |
+----------------+----------------+-------------------+--------+

Multiple entries concatenated:
[Entry1][Entry2][Entry3]...
```

---

### `read_file(path)`

**Purpose:**  
Reads the entire contents of a file from disk as bytes.

**Parameters:**  
- `path` (`str`): File system path to the file.

**Operation Steps:**  
1. Opens the file at the specified path in binary read mode.  
2. Reads all bytes from the file.  
3. Returns the bytes read.

**Example Usage:**

```python
content = read_file('.git/HEAD')
print(content.decode())
```

---

### `cat_file(mode, sha1_prefix)`

**Purpose:**  
Outputs contents or information about a Git object identified by a SHA-1 prefix, supporting various display modes.

**Parameters:**  
- `mode` (`str`): Display mode; one of `'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, or `'pretty'`.  
- `sha1_prefix` (`str`): SHA-1 prefix of the object.

**Operation Steps:**  
1. Reads the object type and data via `read_object(sha1_prefix)`.  
2. Depending on `mode`:  
   - For `'commit'`, `'tree'`, `'blob'`: verifies object type matches mode, then writes raw data to stdout.  
   - For `'size'`: prints the size of the object in bytes.  
   - For `'type'`: prints the object type.  
   - For `'pretty'`:  
     - For `'commit'` and `'blob'`, prints raw data.  
     - For `'tree'`, parses tree entries and prints each entry with mode, type (`tree` or `blob`), SHA-1, and path in a formatted manner.  
3. Raises `ValueError` for unsupported modes or mismatched object types.

**Example Usage:**

```python
# Print prettified tree object contents
cat_file('pretty', '4b825dc642cb6eb9a060e54bf8d69288fbee4904')
```

**Sample Output:**

```
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    README.md
040000 tree 3f825dc642cb6eb9a060e54bf8d69288fbee4933    src
```

---

## Additional Notes

- These functions rely heavily on the `.git/objects` directory structure, where objects are stored using the first two characters of their SHA-1 as the directory name, and the remaining characters as the filename.  
- Objects are stored compressed with zlib, and the internal format includes a header (`<type> <size>\0`) followed by the raw data bytes.  
- Tree objects represent directories and contain multiple entries, each with mode, path, and SHA-1, allowing recursive parsing for nested trees.  
- The `cat_file` function is a versatile tool similar to the `git cat-file` command, useful for inspecting Git objects in various formats.

---

## ASCII Diagram: Git Object Storage Layout

```
.git/
└── objects/
    ├── ab/
    │   └── cdef1234567890abcdef1234567890abcdef12
    ├── 4b/
    │   └── 825dc642cb6eb9a060e54bf8d69288fbee4904
    └── ...
```

- The first two characters of the SHA-1 (`ab`, `4b`, etc.) form the directory name.
- The remaining 38 characters form the filename.
- Objects are stored compressed in these files.

---

This concludes the detailed technical reference for reading and writing Git objects, including mechanisms to access and parse tree objects and resolve objects by SHA-1 prefixes.