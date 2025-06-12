# object_storage.md

# Object Storage Operations

## Overview

This document details the core object storage operations pivotal to managing Git objects within the repository. It covers functions responsible for hashing data into Git objects (`hash_object`), reading objects from the object store (`read_object`), and writing raw data to files (`write_file`). These functions underpin the functionality of the Git object database, enabling storage, retrieval, and integrity verification of Git objects such as blobs, trees, and commits.

Situated within the larger "Object Handling and Git Objects" section, this file focuses on low-level data operations critical for commit creation, repository state management, and data integrity. Understanding these functions is essential for grasping how the repository stores versioned content efficiently and securely.

---

## Function Documentation

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash of the Git object constructed from the input `data` and its type (`obj_type`), optionally write the compressed object to the Git object store, and return the hexadecimal SHA-1 hash string.

**Parameters:**  
- `data` (`bytes`): Raw content to be stored as a Git object (e.g., file contents for blobs).  
- `obj_type` (`str`): The Git object type, such as `'blob'`, `'tree'`, or `'commit'`.  
- `write` (`bool`, optional): If `True` (default), writes the compressed object to disk under `.git/objects`. If `False`, only computes the hash without storing the object.

**Operation:**  
1. Construct the Git object header by concatenating the object type, a space, and the length of the data (in bytes), then encode to bytes.  
2. Append a null byte (`\x00`) after the header.  
3. Concatenate the header and the actual data to form the full object content.  
4. Compute the SHA-1 hash of this full data.  
5. If `write` is enabled, compress the full data using zlib and store it on disk under `.git/objects/xx/yyyyyyyy...`, where `xx` is the first two hex digits of the hash and `yyyyyyyy...` is the remaining 38 hex digits. If the object already exists, skip writing.  
6. Return the SHA-1 hash as a hexadecimal string.

**Example Usage:**

```python
file_contents = b'Hello, Git!'
sha1_hash = hash_object(file_contents, 'blob')
print("Stored blob object with hash:", sha1_hash)
```

---

### `write_file(path, data)`

**Purpose:**  
Write raw byte data to a file at the specified file system path.

**Parameters:**  
- `path` (`str`): The file path where the data will be written.  
- `data` (`bytes`): The byte content to write to the file.

**Operation:**  
1. Open the file at `path` in binary write mode (`'wb'`).  
2. Write the `data` bytes to the file.  
3. Close the file handle automatically upon exiting the context.

**Example Usage:**

```python
path = '.git/refs/heads/master'
commit_hash = b'abc123def4567890\n'
write_file(path, commit_hash)
print(f"Wrote commit hash to {path}")
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Retrieve a Git object from the object store by its SHA-1 prefix, decompress it, and return its type and raw data.

**Parameters:**  
- `sha1_prefix` (`str`): The partial or full SHA-1 hash prefix to identify the object.

**Returns:**  
A tuple `(object_type, data_bytes)` where:  
- `object_type` (`str`): The type of the object (e.g., `'blob'`, `'tree'`, `'commit'`).  
- `data_bytes` (`bytes`): The raw data content of the object.

**Preconditions:**  
- The object identified by `sha1_prefix` must exist in `.git/objects`.  
- The prefix must be unambiguous.

**Operation:**  
1. Locate the full path of the object file by resolving the SHA-1 prefix to the exact object file path.  
2. Read the compressed object data from the file.  
3. Decompress the data using zlib.  
4. Parse the header to extract the object type and size.  
5. Extract the raw data following the null byte separator.  
6. Verify the data length matches the size declared in the header.  
7. Return the object type and data.

**Example Usage:**

```python
object_type, data = read_object('e69de29')
print(f"Object type: {object_type}")
print("Data content:", data)
```

---

## ASCII Diagram: Object Storage Layout

```
.git/
│
├── objects/
│   ├── xx/                    # first two hex digits of SHA-1
│   │   ├── yyyyyyyyyyyyyyyyy  # remaining 38 hex digits as filename
│   │   └── ...
│   └── ...
├── refs/
│   └── heads/
│       └── master             # file storing current master commit SHA-1
└── ...
```

- Objects are stored in subdirectories named after the first two characters of their SHA-1 hash.
- The remaining 38 characters form the filename.
- Each object file contains a compressed Git object (header + data).

---

This documentation provides a clear understanding of how Git objects are hashed, stored, and retrieved at the filesystem level, forming the foundation for higher-level Git operations such as commits, trees, and branches.