# object_hashing.md

## Overview

This document explains the hashing mechanism for Git objects within the `pygit` repository implementation, focusing on the `hash_object` function and the related object storage process. Git objects such as blobs, trees, and commits are uniquely identified by their SHA-1 hash, computed from their content and type. This file details how these hashes are computed, stored in the `.git/objects` directory, and how the integrity and uniqueness of objects are maintained. The document is part of the "Index and Objects" section, which provides essential details on how Git internally handles object and index management. Understanding object hashing is fundamental to grasping how Git tracks content efficiently and ensures data integrity.

---

## Functions

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of Git object data of a specified type (`blob`, `tree`, `commit`, etc.) and optionally stores the compressed object in the Git object database. Returns the SHA-1 hash as a hex string.

**Parameters:**

- `data` (`bytes`): The raw content of the object.
- `obj_type` (`str`): The Git object type, e.g., `"blob"`, `"tree"`, `"commit"`.
- `write` (`bool`, optional): If `True`, writes the object to the `.git/objects` directory after compressing it. Defaults to `True`.

**Operation:**

1. Constructs a header string of the form:  
   ```
   "{obj_type} {len(data)}"
   ```
2. Combines the header, a null byte (`\x00`), and the data into a single byte sequence:
   ```
   full_data = header + b'\x00' + data
   ```
3. Computes the SHA-1 hash of `full_data`.
4. If `write` is `True`, stores the compressed `full_data` in the object store under `.git/objects/XX/YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY` where `XX` are the first two hex digits of the hash and `YYYY...` the remaining 38.
5. Returns the SHA-1 hash hex string.

**Example Usage:**

```python
# Read file contents
content = read_file('README.md')

# Hash as a blob object and write to object store
sha1_hash = hash_object(content, 'blob')

print(f"Object stored with SHA-1: {sha1_hash}")
```

---

### How Objects Are Stored

Git stores objects in the `.git/objects` directory in a two-level subdirectory structure. The first two hex characters of the SHA-1 hash form the directory name, and the remaining 38 characters form the filename. Objects are stored compressed using zlib.

**ASCII Diagram:**

```
.git/
└── objects/
    ├── 1a/
    │   └── e4c3b2f1d4... (compressed object file)
    ├── 7f/
    │   └── 9b6d2e3f4a...
    └── ...
```

When `hash_object` writes an object, it:

- Creates the directory if it does not exist.
- Compresses the full object data.
- Writes the compressed data to the file named after the SHA-1 hash suffix.

---

### Additional Context and Related Functions

While `hash_object` handles the hashing and storage of objects, other functions interact with the stored objects:

- `read_object(sha1_prefix)`: Reads and decompresses an object by SHA-1 prefix, returning the object type and raw data.
- `find_object(sha1_prefix)`: Locates the path to an object file given a SHA-1 prefix.
- `write_file(path, data)`: Writes byte data to a file (used internally by `hash_object`).
- `read_file(path)`: Reads file content as bytes.
- `write_tree()`: Writes tree objects constructed from the current index entries.
- `commit(message, author=None)`: Creates commit objects referencing tree objects and parents.

---

## Detailed Function Documentation

---

### `hash_object(data, obj_type, write=True)`

```python
def hash_object(data, obj_type, write=True):
    """Compute hash of object data of given type and write to object store if
    'write' is True. Return SHA-1 object hash as hex string.
    """
    header = '{} {}'.format(obj_type, len(data)).encode()
    full_data = header + b'\x00' + data
    sha1 = hashlib.sha1(full_data).hexdigest()
    if write:
        path = os.path.join('.git', 'objects', sha1[:2], sha1[2:])
        if not os.path.exists(path):
            os.makedirs(os.path.dirname(path), exist_ok=True)
            write_file(path, zlib.compress(full_data))
    return sha1
```

**Step-by-step Explanation:**

1. **Create header:** Prepare a header with the format `<obj_type> <size>` in bytes.
2. **Concatenate data:** Combine header, a null byte, and the actual data to form the full object content.
3. **Hash computation:** Calculate SHA-1 hash over the full object content.
4. **Write to object store:** If `write` is enabled, ensure the directory for the object exists, compress the full content with zlib, and store it.
5. **Return:** Provide the SHA-1 hash of the object as a string.

---

### Usage Example

```python
from pygit import read_file, hash_object

# Load file content
file_data = read_file('example.txt')

# Hash and store as a blob object
blob_sha1 = hash_object(file_data, 'blob')

print(f"Blob object stored with SHA-1: {blob_sha1}")
```

---

## Summary Diagram of Object Hashing Flow

```
+-----------------+         +---------------------+         +----------------------------+
| Raw object data  |  --->   | Prepare header + data|  --->   | Compute SHA-1 hash          |
+-----------------+         +---------------------+         +----------------------------+
                                                                      |
                                                                      v
                                                          +----------------------------+
                                                          | Store compressed object in  |
                                                          | .git/objects/XX/YYYY...     |
                                                          +----------------------------+
                                                                      |
                                                                      v
                                                          +----------------------------+
                                                          | Return SHA-1 hash string    |
                                                          +----------------------------+
```

---

## Conclusion

The `hash_object` function is central to how `pygit` manages Git objects. By hashing the content with a type-specific header and storing compressed data in a structured directory layout, `pygit` mirrors Git's fundamental content-addressable storage mechanism. This approach ensures efficient storage, deduplication, and integrity verification of repository data.

This file complements other documentation on index handling, commit creation, and object reading to provide a full understanding of Git's internal data management in the `pygit` project.