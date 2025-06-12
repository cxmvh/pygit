# read_object.md

# Reading Git Objects by SHA-1 Prefix

---

## Overview

The `read_object.md` document provides detailed guidance on reading Git objects using a SHA-1 prefix. It explains how Git stores its data in compressed object files identified by their SHA-1 hashes and how to retrieve and decompress these objects to access their type and content. This file fits within the broader "Object Handling and Git Objects" section of the documentation tree, which focuses on managing Git objects including blobs, trees, commits, and packfiles. Understanding how to read objects by their SHA-1 prefix is essential for implementing core Git functionalities such as diffing, status reporting, and commit operations, as these rely on accessing object data efficiently and correctly.

---

## Function Documentation

### `read_object(sha1_prefix)`

#### Purpose

Reads a Git object from the repository given a SHA-1 prefix. Returns a tuple containing the object's type (e.g., `'blob'`, `'tree'`, `'commit'`) and the raw data bytes of the object. This function raises a `ValueError` if the object cannot be found or if the prefix is ambiguous.

#### Parameters

- `sha1_prefix` (str): A string representing the prefix of the SHA-1 hash identifying the Git object. Typically at least 2 characters long.

#### Preconditions

- The SHA-1 prefix must be at least 2 characters long.
- The `.git` directory must exist and contain the object files.
- The object with the given prefix must exist and be unique.

#### How it works

1. **Locate the object file**  
   Calls `find_object(sha1_prefix)` to find the full object file path in the `.git/objects` directory corresponding to the prefix.

2. **Read and decompress the object data**  
   Reads the compressed contents of the object file using `read_file(path)`. The data is then decompressed using `zlib.decompress`.

3. **Parse the object header**  
   The decompressed data starts with a header of the form:  
   ```
   <object_type> <size>\x00
   ```  
   The function locates the null byte (`\x00`) separator to extract the header.

4. **Extract object type and size**  
   The header is decoded and split into the object type string (e.g., `'blob'`) and size (number of bytes in the data).

5. **Validate the data size**  
   The remainder of the decompressed data after the header is the actual object content. The function asserts that the size matches the length of this data, ensuring data integrity.

6. **Return result**  
   Returns a tuple `(object_type, data)` where `data` is the raw bytes of the object content.

#### Example Usage

```python
from pygit import read_object

# Read the object with SHA-1 prefix 'a1b2c3'
obj_type, data = read_object('a1b2c3')

print(f"Object type: {obj_type}")
print(f"Data (first 100 bytes): {data[:100]!r}")
```

If the prefix does not uniquely identify an object or if the object is not found, a `ValueError` will be raised.

---

### Supporting Function: `find_object(sha1_prefix)`

*(Summarized for context)*

- Finds the full path to the Git object file based on the SHA-1 prefix.
- Ensures the prefix is at least 2 characters.
- Looks up the `.git/objects/xx/` directory where `xx` is the first two characters of the prefix.
- Matches the remainder of the prefix to find the unique object filename.
- Raises errors if no objects or multiple objects match the prefix.

---

## ASCII Diagram: Git Object Storage and Lookup

```
.git/
└── objects/
    ├── aa/
    │   ├── 1b2c3d4e5f6g7h8i9j0k...
    │   └── ...
    ├── b1/
    │   ├── c3d4e5f6g7h8i9j0k1l2...
    │   └── ...
    └── ...

SHA-1 hash: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0

Lookup steps for prefix 'a1b2c3':
1. Directory: '.git/objects/a1/'
2. Files: list all files starting with 'b2c3'
3. Find unique match, e.g., 'b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0'
4. Read and decompress object contents
```

---

## Additional Notes

- The `read_object` function is fundamental for many operations like showing diffs, status, and commits because it allows retrieval of the exact Git object data from the object store.
- The use of SHA-1 prefixes improves usability, letting users specify just enough characters to uniquely identify an object.

---

# Summary

The `read_object` function is a critical utility in Git object handling, enabling precise reading of object content identified by SHA-1 prefixes. Combined with the object-finding logic and low-level file reads, it forms the backbone of accessing Git's internal data structures. This documentation should assist developers in understanding and using this function effectively within the larger Git implementation.