# Object Storage

## Overview

This document details the core object storage mechanisms used in the repository, specifically focusing on hashing, writing, and reading git objects. It covers three primary functions: `hash_object`, `write_file`, and `read_object`, which collectively enable storing raw data as Git objects, persisting them efficiently, and retrieving them when needed. This file fits within the broader context of repository initialization and setup, providing foundational utilities that underpin object management throughout the repository lifecycle.

---

## Function Documentation

### `hash_object(data, obj_type, write=True)`

#### Purpose

Computes the SHA-1 hash of an object of a specified type (`blob`, `tree`, `commit`, etc.) from the provided raw `data`. Optionally writes the compressed object data to the `.git/objects` directory if `write` is `True`. Returns the SHA-1 object hash as a hexadecimal string.

#### Parameters

- `data` (`bytes`): Raw byte content of the object to be hashed.
- `obj_type` (`str`): The Git object type (e.g., `'blob'`, `'tree'`, `'commit'`).
- `write` (`bool`, optional): Whether to write the object to storage. Defaults to `True`.

#### Operation Details

1. Construct the object header in the format: `"{obj_type} {len(data)}"`.
2. Concatenate the header, a null byte (`\x00`), and the data to form the full object content.
3. Compute the SHA-1 hash of the full content.
4. If `write` is `True`:
   - Derive the path in `.git/objects` using the first two hex digits as a directory and the remaining as filename.
   - If the object does not already exist at this path:
     - Create necessary directories.
     - Compress the full content using zlib.
     - Write the compressed data to the file.
5. Return the SHA-1 hash string.

#### Example Usage

```python
# Hash and store a blob object containing file content
file_data = b'Hello, Git object storage!'
sha1_hash = hash_object(file_data, 'blob')
print(f"Stored object hash: {sha1_hash}")
```

#### ASCII Diagram: Object Storage Layout

```
.git/objects/
    ├── 1a/
    │    └── 2b3c4d5e6f7g8h9i0jklmnopqrstuvwx  # object file named by SHA-1 suffix
    ├── 5f/
    │    └── a8b7c6d5e4f3g2h1ijklmnopqrstuvwx
    └── ...
```

---

### `write_file(path, data)`

#### Purpose

Writes raw bytes data to a specified file path. It is a low-level utility function used internally to persist data such as compressed git objects or repository metadata.

#### Parameters

- `path` (`str`): The file system path where data should be written.
- `data` (`bytes`): The byte content to write to the file.

#### Operation Details

1. Open the target file in binary write mode (`'wb'`).
2. Write the entire `data` bytes to the file.
3. Close the file automatically via context management.

#### Example Usage

```python
# Write arbitrary data to a file in the repository
write_file('.git/HEAD', b'ref: refs/heads/master')
```

---

### `read_object(sha1_prefix)`

#### Purpose

Retrieves and decompresses a git object from the object store using a SHA-1 prefix, returning a tuple consisting of the object type and the raw data bytes. Raises a `ValueError` if the object cannot be found or if the decompression or format validation fails.

#### Parameters

- `sha1_prefix` (`str`): The prefix (partial or full) of the SHA-1 hash identifying the object.

#### Operation Details

1. Locate the full object file path using the prefix (by searching `.git/objects` directories).
2. Read and decompress the zlib-compressed file contents.
3. Extract the object header, which includes:
   - Object type (e.g., `blob`, `commit`, etc.).
   - Size of the decompressed data.
4. Verify that the declared size matches the actual data length.
5. Return a tuple: `(object_type, data_bytes)`.

#### Example Usage

```python
# Read an object from the store by its SHA-1 prefix
obj_type, content = read_object('1a2b3c4d5e')
print(f"Object type: {obj_type}")
print(f"Content bytes: {content}")
```

#### ASCII Diagram: Object Read Flow

```
sha1_prefix: "1a2b3c..."

.git/objects/
  └── 1a/
       └── 2b3c4d5e6f7g8h9i0jklmnopqrstuvwx  <-- compressed object data file

  read_file(path) --(zlib.decompress)--> full_data
  full_data = b"{obj_type} {size}\x00{data}"

  Extract:
   +-----------+------------+-------------+
   | obj_type  | size bytes | data bytes  |
   +-----------+------------+-------------+
```

---

# Summary

This document provides essential insight into how raw data is converted into Git objects, stored efficiently, and retrieved for operations in the repository. The three functions described (`hash_object`, `write_file`, and `read_object`) form the foundational primitives enabling the object storage layer crucial for repository integrity and functionality.