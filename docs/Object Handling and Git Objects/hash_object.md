# hash_object.md

# Hashing Data and Storing as Git Objects

---

## Overview

The `hash_object.md` file documents the functionality related to hashing raw data and storing it as Git objects within the `.git/objects` directory. This process is fundamental to Git's content-addressable storage model: any data (such as file contents, trees, or commits) is converted into a SHA-1 hash, which serves as a unique identifier for that data. The file is part of the broader **Object Handling and Git Objects** section in the documentation tree, which covers reading, writing, hashing, storing, and managing Git objects including blobs, trees, commits, and packfiles.

This functionality is especially significant because it underpins many core Git operations like staging files, committing changes, and transferring objects between repositories. By understanding how data is hashed and stored, one gains insight into Git's integrity guarantees and efficient storage mechanisms.

---

## Function Documentation

### `hash_object(data, obj_type, write=True)`

#### Purpose

Compute the SHA-1 hash of given data with an associated Git object type (e.g., `blob`, `tree`, `commit`). Optionally write the object to the Git object store on disk.

- **`data`**: Raw bytes of the content to hash (e.g., file content for blobs).
- **`obj_type`**: The type of Git object as a string (`'blob'`, `'tree'`, `'commit'`, etc.).
- **`write`**: If `True`, store the compressed object in the `.git/objects` directory. If `False`, only compute and return the hash without storing.

Returns the SHA-1 hash as a 40-character hexadecimal string.

#### How It Works

1. Construct an object header string combining the object type and size of the data, e.g., `"blob 14"`.
2. Concatenate the header, a null byte (`\x00`), and the raw data bytes.
3. Compute the SHA-1 hash of this combined data.
4. If `write` is `True`, compress the full data using zlib and write it to an object file under `.git/objects/XX/YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY`, where `XX` are the first two hash characters and `YY...` are the remaining 38 characters.
5. Return the SHA-1 hash string.

#### Example Usage

```python
# Read file contents
file_data = read_file('example.txt')

# Hash file contents as a blob object and write to object store
sha1_hash = hash_object(file_data, 'blob')

print(f"Stored blob object with SHA-1: {sha1_hash}")
```

---

## Additional Details and Context

### Git Object Storage Layout

Git stores objects in a directory structure based on the SHA-1 hash:

```
.git/objects/
├── 12/
│   └── 3456789abcdef... (compressed object data)
├── ab/
│   └── cdef1234567890... (compressed object data)
└── ...
```

The first two characters of the SHA-1 hash form the directory name, and the remaining 38 characters form the filename.

---

## ASCII Diagram: Object Storage Path

```
SHA-1:  1234567890abcdef1234567890abcdef12345678
        || 
        ||---> Directory: .git/objects/12/
                           └── File: 34567890abcdef1234567890abcdef12345678
```

This structure helps keep the filesystem efficient by avoiding too many files in a single directory.

---

## Related Functions

While `hash_object` focuses on hashing and storing raw data into Git's object store, it integrates closely with other functions:

- `read_file(path)`: Reads file contents as bytes.
- `write_file(path, data)`: Writes bytes to a file.
- `find_object(sha1_prefix)`: Finds object file paths from SHA-1 prefixes.
- `read_object(sha1_prefix)`: Reads and decompresses Git objects by SHA-1 prefix.
- `add(paths)`: Adds files to the index by hashing blobs.
- `write_tree()`: Creates tree objects from index entries.
- `commit(message)`: Creates commit objects referencing trees.

---

## Summary

The `hash_object` function is a core utility for converting arbitrary data into Git objects, computing their unique SHA-1 hash, and optionally persisting them in the Git object store. This mechanism provides the foundation for Git’s data integrity, deduplication, and efficient storage model.

---

# Full Reference Code for `hash_object`

```python
import os
import hashlib
import zlib

def hash_object(data, obj_type, write=True):
    """Compute hash of object data of given type and write to object store if
    "write" is True. Return SHA-1 object hash as hex string.
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

def write_file(path, data):
    """Write data bytes to file at given path."""
    with open(path, 'wb') as f:
        f.write(data)
```

---

# End of `hash_object.md` Documentation