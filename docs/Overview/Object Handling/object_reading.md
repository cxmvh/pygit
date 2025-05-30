# object_reading.md

# Overview

This document details the functions responsible for reading and locating Git objects within the repository, primarily focusing on `read_object` and `find_object`. These functions are fundamental to the Git object management system, enabling retrieval of Git objects by their SHA-1 hashes, verifying their integrity, and decoding their contents. This file fits under the **Object Handling** section of the documentation tree and is closely related to the `pygit.cat_file` execution flow, where object inspection and display are performed. Understanding these functions is vital for operations that require accessing Git objects, such as inspecting commits, trees, blobs, or performing diffs.

---

# Function Documentation

## `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object identified by a SHA-1 prefix and returns its type and decompressed data.

**Parameters:**  
- `sha1_prefix` (str): A hexadecimal string prefix of the full SHA-1 hash identifying the object.

**Returns:**  
- Tuple `(obj_type, data)` where:
  - `obj_type` (str): The type of the Git object (e.g., `'commit'`, `'tree'`, `'blob'`).
  - `data` (bytes): The raw data content of the object.

**Preconditions:**  
- The prefix must uniquely identify a single Git object in the `.git/objects` directory.
- The object must exist and be accessible.

**Operation Steps:**  
1. Calls `find_object(sha1_prefix)` to locate the full path of the object file.
2. Reads the compressed object file content using `read_file(path)`.
3. Decompresses the file content using `zlib.decompress`.
4. Splits the decompressed data into a header and data content at the first null byte (`\x00`).
5. Parses the header to extract the object type and expected size of the data.
6. Confirms that the size matches the actual data length.
7. Returns the object type and data.

**Example Usage:**

```python
obj_type, data = read_object("a1b2c3d")
print(f"Object type: {obj_type}")
print(f"Object data (first 100 bytes): {data[:100]!r}")
```

---

## `find_object(sha1_prefix)`

**Purpose:**  
Finds the file path of a Git object given a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (str): A hexadecimal string prefix (at least 2 characters) of the SHA-1 object hash.

**Returns:**  
- `path` (str): The full path to the object file in the `.git/objects` directory.

**Preconditions:**  
- `sha1_prefix` must be at least 2 characters.
- Exactly one object must match the prefix; otherwise, an error is raised.

**Operation Steps:**  
1. Validates that the prefix length is at least 2 characters.
2. Constructs the directory path using the first two characters of the prefix.
3. Lists files in that directory and filters those whose names start with the remaining characters of the prefix.
4. If no matching objects are found, raises `ValueError`.
5. If multiple matching objects are found, raises `ValueError` indicating ambiguity.
6. Returns the path to the uniquely identified object file.

**Example Usage:**

```python
try:
    object_path = find_object("a1b2c3d")
    print(f"Object found at: {object_path}")
except ValueError as e:
    print(f"Error finding object: {e}")
```

---

# ASCII Diagram: Git Object Storage Layout

```
.git/
└── objects/
    ├── aa/
    │   ├── 1b2c3d4e5f6g7h8i9j0k...
    │   └── ...
    ├── bb/
    │   └── 1234567890abcdef...
    └── ...
```

- The first two characters of the SHA-1 hash identify a subdirectory inside `.git/objects`.
- The remaining 38 characters form the file name inside that directory.
- `find_object` uses this structure to locate objects by prefix.

---

## Related Function: `cat_file(mode, sha1_prefix)`

Although not the primary focus, `cat_file` uses `read_object` to output object data or metadata based on the mode provided.

**Example:**

```python
cat_file('type', 'a1b2c3d')
# Prints the object type for the given SHA-1 prefix.
```

---

# Summary

The `read_object` and `find_object` functions provide the essential mechanism for locating and reading raw Git objects stored in the `.git/objects` directory. They form the backbone of object inspection workflows like `cat_file`, enabling higher-level commands and utilities to process Git data accurately and efficiently. These functions ensure data integrity and resolve ambiguities in object identification by SHA-1 prefixes.