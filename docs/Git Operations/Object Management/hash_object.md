# hash_object.md

# Hashing and Storing Git Objects in the Object Database

## Overview

This document details the functionality for hashing and storing Git objects within the `.git/objects` database in the pygit implementation. The `hash_object` function is central to this process, responsible for computing the SHA-1 hash of the object data, optionally writing the compressed object to the object store, and returning the object's SHA-1 hash string.

This file is part of the **Object Management** section in the overall pygit documentation tree. It supports key operations such as committing changes, pushing to remote repositories, and managing the object database. Proper hashing and storage of Git objects (blobs, trees, commits) are essential to Git's content-addressed storage model, ensuring data integrity and efficient object retrieval.

---

## Function: `hash_object(data, obj_type, write=True)`

### Purpose

Compute the SHA-1 hash of an object with given data and type. Optionally write the compressed object to the Git object store under the `.git/objects` directory.

This function is critical for:

- Creating blob objects from file contents.
- Creating tree objects from directory listings.
- Creating commit objects representing snapshots.
- Ensuring objects are stored in a content-addressable manner.

### Parameters

- **data** (`bytes`): The raw content bytes of the object.
- **obj_type** (`str`): The type of the object. Common types include `'blob'`, `'tree'`, and `'commit'`.
- **write** (`bool`, default `True`): Whether to write the object data to the object database. If `False`, only compute and return the hash without storing.

### Returns

- (`str`): The SHA-1 object hash as a hexadecimal string.

### Detailed Operation

1. **Prepare Object Data:**

   - Construct the object header as a byte string: `"{obj_type} {len(data)}"`.
   - Concatenate the header, a null-byte (`b'\x00'`), and `data` to form the full object content.

2. **Compute SHA-1 Hash:**

   - Compute the SHA-1 hash of the full object content (header + null byte + data).
   - This hash acts as a unique identifier for the object.

3. **Optionally Write Object:**

   - If `write` is `True`, determine the storage path:
     - Directory: `.git/objects/` plus the first two characters of the SHA-1.
     - Filename: remaining 38 characters of the SHA-1.
   - If the object file does not already exist:
     - Create necessary directories.
     - Compress the full object content using zlib.
     - Write the compressed data to the object file.

4. **Return the SHA-1 Hash:**

   - Return the hexadecimal SHA-1 string.

### Usage Example

```python
# Read file contents as bytes
file_data = read_file('README.md')

# Hash as a blob object and store in object database
sha1_hash = hash_object(file_data, 'blob')

print(f'Blob object SHA-1: {sha1_hash}')
```

### Notes

- The object store structure uses the first two characters of the SHA-1 as a directory name, improving filesystem efficiency.
- Writing is skipped if the object already exists, avoiding redundant storage.
- The function supports creating other object types by changing `obj_type` and `data` accordingly.

---

## Related Functions and Concepts

To fully understand the role of `hash_object`, it is helpful to consider its relationship with other functions:

- **`read_object(sha1_prefix)`**: Reads and decompresses an object by its SHA-1 hash prefix, returning the object type and data.
- **`write_file(path, data)`**: Utility to write bytes to a file path, used internally by `hash_object`.
- **`find_object(sha1_prefix)`**: Locates the path of an object file given its SHA-1 prefix.
- **`commit(message, author)`**: Uses `hash_object` to create commit objects.
- **`write_tree()`**: Uses `hash_object` to create tree objects from index entries.
- **`add(paths)`**: Creates blob objects for files and updates the index.
- **`push(git_url, username, password)`**: Relies on object hashing and packing to push commits to remotes.

---

## ASCII Diagram: Object Hashing and Storage Flow

```
+-------------------+        +-------------------------+
| Raw Object Data    |        | Object Header:          |
| (file content,     |        | "{obj_type} {size}"     |
|  tree entries,     |  +-->  | + Null byte (0x00)      |
|  commit info)      |        +-------------------------+
+-------------------+                |
                                      v
                            +-------------------------+
                            | Concatenate Header +    |
                            | Null Byte + Data Bytes  |
                            +-------------------------+
                                      |
                                      v
                            +-------------------------+
                            | Compute SHA-1 Hash of   |
                            | Full Object Content     |
                            +-------------------------+
                                      |
                                      v
       +------------------------+          +-----------------------------+
       | Object Exists in Store? | --Yes--> | Return SHA-1 Hash            |
       +------------------------+          +-----------------------------+
               | No
               v
       +------------------------+
       | Compress Object Data    |
       +------------------------+
               |
               v
       +------------------------+
       | Write to .git/objects/ |
       | [first 2 chars]/       |
       | [remaining 38 chars]   |
       +------------------------+
               |
               v
       +------------------------+
       | Return SHA-1 Hash      |
       +------------------------+
```

---

## Summary

The `hash_object` function implements the core Git mechanism of content-addressed storage by hashing object data and storing compressed object files in the `.git/objects` directory. This process underpins the integrity and efficiency of the Git object database, enabling the tracking of file contents, directory trees, and commit history in a reliable, versioned manner.