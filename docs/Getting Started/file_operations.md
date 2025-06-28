# file_operations.md

# File Operations: Reading and Writing Files in pygit

## Overview

This document covers essential file input/output functions used within the pygit repository management tool. Specifically, it details the `read_file` and `write_file` functions, which are fundamental utilities for reading bytes from files and writing bytes to files respectively.

These functions form the basic building blocks for more complex operations like managing git objects, index files, commit data, and repository initialization. As part of the "Getting Started" section of the pygit documentation, they support other key operations such as `init` (repository initialization) and `push` (uploading commits to remote repositories).

---

## Function: read_file(path)

### Purpose

Reads the entire contents of a file at the specified path and returns it as a bytes object. This function is used wherever raw file data is needed, such as reading git objects, index files, or working directory files.

### Parameters

- **path** (`str`): The file system path to the file to be read.

### Behavior

- Opens the file in binary read mode (`'rb'`).
- Reads all contents of the file.
- Returns the data as bytes.
- Raises `FileNotFoundError` if the specified file does not exist.

### Usage Example

```python
# Read the contents of a git object file
object_data = read_file('.git/objects/ab/cdef1234...')

# Read the index file contents
index_data = read_file('.git/index')
```

---

## Function: write_file(path, data)

### Purpose

Writes the given bytes data to a file at the specified path. This function is used to save git objects, index files, references, and other repository data.

### Parameters

- **path** (`str`): The file system path where the data should be written.
- **data** (`bytes`): The data to write into the file.

### Behavior

- Opens the file in binary write mode (`'wb'`).
- Writes all bytes from `data` to the file.
- Overwrites the file if it already exists.
- Creates any necessary directories beforehand (handled externally if needed).

### Usage Example

```python
# Write compressed git object data to object database
write_file('.git/objects/ab/cdef1234...', compressed_object_data)

# Write HEAD reference file to point to master branch
write_file('.git/HEAD', b'ref: refs/heads/master')
```

---

## Integration with Other Functions

Both `read_file` and `write_file` are extensively used throughout the pygit codebase. For example:

- `init(repo)` creates necessary git directories and writes initial files using `write_file`.
- `hash_object(data, obj_type, write=True)` compresses and stores git objects using `write_file`.
- `read_index()` reads the git index file using `read_file`.
- `write_index(entries)` serializes and writes index entries using `write_file`.
- `commit(message, author=None)` writes commit objects and updates refs using `write_file`.
- `get_local_master_hash()` reads the current HEAD commit hash using `read_file`.

### Example Usage Within `init(repo)`

```python
def init(repo):
    """Create directory for repo and initialize .git directory."""
    os.mkdir(repo)
    os.mkdir(os.path.join(repo, '.git'))
    for name in ['objects', 'refs', 'refs/heads']:
        os.mkdir(os.path.join(repo, '.git', name))
    # Write HEAD file pointing to master branch
    write_file(os.path.join(repo, '.git', 'HEAD'), b'ref: refs/heads/master')
    print('initialized empty repository: {}'.format(repo))
```

---

## ASCII Diagram: File Operations in pygit

```
+-----------------------+
|   File Path: 'path'   |
+-----------+-----------+
            |
            v
+-----------------------+     read_file(path)      +----------------------+
| Opens file in 'rb'    | -----------------------> | Returns file contents |
| Reads all bytes       |                           +----------------------+
+-----------------------+

+-----------------------+
|   File Path: 'path'   |
|   Data: bytes         |
+-----------+-----------+
            |
            v
+-----------------------+     write_file(path, data) +--------------------+
| Opens file in 'wb'    | -------------------------> | Writes all bytes to |
| Writes all bytes      |                            | file at 'path'      |
+-----------------------+                            +--------------------+
```

---

## Summary

The `read_file` and `write_file` functions are minimal yet critical utilities in pygit for handling byte-level file I/O. They abstract away file opening modes and ensure consistent reading and writing of repository data, enabling higher-level git operations to function correctly and reliably.