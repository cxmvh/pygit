# hash_object.md

# Hash Object: Hashes Data to Create Git Objects and Optionally Write Them

---

## Overview

The `hash_object` module provides functionality to hash arbitrary data into Git object format and optionally write these objects into the Git object store (`.git/objects`). This is a fundamental operation in Git's internal storage mechanism, enabling the creation of blobs, trees, commits, and other objects by computing their SHA-1 hashes and saving their compressed representations.

This file belongs to the **Object Handling** section within the broader Git implementation documentation. It complements related files like `write_file.md` (for file writing utilities) and integrates closely with object reading and indexing functionalities documented elsewhere. The `hash_object` function plays a critical role in commands like adding files to the index, writing trees, and committing changes by generating the necessary Git objects to represent the repository state.

---

## Functions

### `hash_object(data, obj_type, write=True)`

Hashes the provided data as a Git object of the specified type and optionally writes the compressed object into the Git object database.

#### Purpose

- Compute the SHA-1 hash of a Git object composed of a header and the provided data.
- Optionally write the compressed object data to the Git object store under `.git/objects`.
- Return the SHA-1 hash as a hexadecimal string, uniquely identifying the object.

#### Parameters

- `data` (`bytes`): The raw content to be stored in the Git object (e.g., file contents for blobs, serialized tree entries for trees).
- `obj_type` (`str`): The Git object type — typically `'blob'`, `'tree'`, `'commit'`, etc.
- `write` (`bool`, optional): If `True` (default), writes the object into the Git object store; if `False`, only returns the SHA-1 without writing.

#### Preconditions

- The `data` must be in bytes format.
- The `obj_type` must be a valid Git object type recognized by the repository.

#### Operation Details

1. Constructs a Git object header by concatenating the `obj_type`, a space, the decimal length of `data`, and a null byte (`\x00`).
2. Concatenates the header and the `data` to form the full object content.
3. Computes the SHA-1 hash of this full content.
4. If `write` is `True`:
   - Determines the storage path in `.git/objects/` using the first two characters of the SHA-1 hash as a directory name, and the remaining characters as the filename.
   - Creates the directory if it does not exist.
   - Compresses the full object content using zlib compression.
   - Writes the compressed data to the determined path.
5. Returns the SHA-1 hash as a hexadecimal string.

#### Usage Example

```python
from pygit import hash_object, read_file

# Read file content to be stored as a blob
file_content = read_file('example.txt')

# Hash the content as a blob and write the object to the store
sha1 = hash_object(file_content, 'blob')
print(f'Blob object created with SHA-1: {sha1}')

# Hash the content without writing to the object store (dry run)
sha1_no_write = hash_object(file_content, 'blob', write=False)
print(f'SHA-1 without writing: {sha1_no_write}')
```

#### ASCII Diagram: Object Storage Path

```
SHA-1: e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
  |       |___________________________|
  |                   |
  |                   +-- filename (38 characters)
  +-- directory (2 characters)

Object path in .git/objects:
.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391
```

This directory structure splits the SHA-1 into a 2-character directory and a 38-character filename, enabling efficient storage and lookup.

---

## Related Functions and Integration Points

- `write_file(path, data)`: Used internally to write compressed object data to disk.
- `read_object(sha1_prefix)`: Reads and decompresses an object from the object store, verifying its type and content.
- `add(paths)`: Uses `hash_object` to hash new blobs before adding entries to the index.
- `write_tree()`: Uses `hash_object` to create tree objects from index entries.
- `commit(message, author)`: Creates commit objects by hashing commit data.

---

By understanding and utilizing `hash_object`, developers and contributors can manipulate Git's internal object database directly, enabling advanced operations such as custom object creation, repository inspection, or low-level manipulation tools.