# Git Index Management (`index.md`)

## Overview

This document details the functions responsible for reading, writing, and updating the Git index, a critical component in Git's internal architecture. The Git index, stored as a binary file in `.git/index`, acts as a staging area between the working directory and the repository's commits. It tracks metadata about files, including their paths, timestamps, sizes, and SHA-1 hashes, ensuring efficient and consistent version control operations.

Located within the "Status and Working Copy Management" section under the "Index Management" subsection, this file provides comprehensive information on handling the Git index and the associated `IndexEntry` data structure. The documented functions support core operations such as adding files to the index, listing indexed files, reading and writing the index file, and facilitating diff computations. These utilities integrate closely with other components like commit creation, status checks, and file listing, underscoring their central role in repository management.

---

## IndexEntry Structure

The `IndexEntry` represents an individual file's metadata within the Git index. Each entry stores file timestamps, device and inode identifiers, permissions, user/group IDs, file size, SHA-1 hash of the file contents, flags, and the file path.

---

## Function Documentation

### 1. `read_index()`

#### Purpose

Reads the Git index file from `.git/index` and returns a list of `IndexEntry` objects representing the indexed files.

#### Parameters

- None

#### Returns

- List of `IndexEntry` objects.

#### Operation Details

- Attempts to read the `.git/index` file; returns an empty list if not found.
- Validates the index file's checksum and signature (`DIRC`).
- Parses the index version (supports version 2).
- Iteratively reads each entry's fixed-length fields and variable-length path.
- Calculates padding to align entries on an 8-byte boundary.
- Returns the complete list of index entries.

#### Example Usage

```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### 2. `write_index(entries)`

#### Purpose

Writes a list of `IndexEntry` objects back to the `.git/index` file, updating the Git index.

#### Parameters

- `entries`: List of `IndexEntry` objects to write.

#### Operation Details

- Packs each entry's fields into binary format.
- Encodes the file path and adds padding to align entries.
- Constructs the index file header with signature, version, and number of entries.
- Computes a SHA-1 checksum for the entire index content.
- Writes the serialized data plus checksum to `.git/index`.

#### Example Usage

```python
write_index(entries)  # entries obtained or modified previously
```

---

### 3. `IndexEntry` (Data Structure)

#### Purpose

Represents metadata for a single file in the Git index.

#### Fields (Typical)

- `ctime_s`, `ctime_n`: Creation time (seconds, nanoseconds)
- `mtime_s`, `mtime_n`: Modification time (seconds, nanoseconds)
- `dev`: Device number
- `ino`: Inode number
- `mode`: File mode (permissions)
- `uid`: User ID
- `gid`: Group ID
- `size`: File size
- `sha1`: SHA-1 hash of file content (20 bytes)
- `flags`: Bit flags including path length
- `path`: File path (string)

---

### 4. `add(paths)`

#### Purpose

Adds one or more file paths to the Git index, updating or inserting entries accordingly.

#### Parameters

- `paths`: List of file paths (strings) to add.

#### Operation Details

- Normalizes paths to use forward slashes.
- Reads current index entries.
- Removes any existing entries matching the input paths.
- For each new path:
  - Reads file content and computes its SHA-1 hash.
  - Retrieves file stat metadata.
  - Creates a new `IndexEntry` with the metadata and hash.
- Sorts all entries by path.
- Writes updated entries back to the index file.

#### Example Usage

```python
add(['src/main.py', 'README.md'])
```

---

### 5. `ls_files(details=False)`

#### Purpose

Lists files recorded in the Git index, optionally showing detailed information.

#### Parameters

- `details` (bool): If `True`, prints mode, SHA-1, stage number, and path; otherwise, prints only paths.

#### Operation Details

- Reads the index entries.
- For each entry:
  - Prints either just the path or extended details.
- The stage number is extracted from the flags field.

#### Example Usage

```python
ls_files()  # Lists only file paths

ls_files(details=True)  # Lists paths with mode, SHA-1, and stage
```

---

### 6. `get_status()`

#### Purpose

Determines the status of files in the working directory relative to the Git index.

#### Parameters

- None

#### Returns

- Tuple of three sorted lists: `(changed_paths, new_paths, deleted_paths)`.

#### Operation Details

- Walks the working directory, ignoring `.git`.
- Builds sets of current filesystem paths and indexed paths.
- Computes:
  - Changed: Paths in both sets with differing SHA-1 hashes.
  - New: Paths in working directory but not in index.
  - Deleted: Paths in index but missing in working directory.

#### Example Usage

```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### 7. `diff()`

#### Purpose

Displays line-by-line differences between files in the index and the working copy.

#### Parameters

- None

#### Operation Details

- Uses `get_status()` to find changed files.
- Reads the index and working copy contents for each changed file.
- Uses `difflib.unified_diff` to generate diff output.
- Prints the diff per file with separators.

#### Example Usage

```python
diff()
```

---

### 8. `write_tree()`

#### Purpose

Creates a Git tree object representing the current staged snapshot in the index.

#### Parameters

- None

#### Returns

- SHA-1 hash (hex string) of the newly written tree object.

#### Operation Details

- Reads index entries.
- Asserts all files are top-level (no directories supported).
- Constructs tree entries as concatenation of mode, path, null byte, and SHA-1.
- Concatenates all entries and hashes as a `tree` object.
- Stores the object in the Git object database.

#### Example Usage

```python
tree_hash = write_tree()
print(f"Tree object created: {tree_hash}")
```

---

### 9. `hash_object(data, obj_type, write=True)`

#### Purpose

Computes the SHA-1 hash for a Git object of a given type and optionally writes it to the object store.

#### Parameters

- `data` (bytes): Object content.
- `obj_type` (str): Git object type (e.g., `'blob'`, `'tree'`, `'commit'`).
- `write` (bool): Whether to write the object to the store.

#### Returns

- SHA-1 hash (hex string) of the object.

#### Operation Details

- Creates a header with object type and size.
- Concatenates header, null byte, and data.
- Computes SHA-1 of the full content.
- If `write` is True, compresses and writes the object to `.git/objects/`.
- Returns the hexadecimal hash.

#### Example Usage

```python
sha1 = hash_object(b'hello world\n', 'blob')
print(f"Object hash: {sha1}")
```

---

### 10. `read_object(sha1_prefix)`

#### Purpose

Reads a Git object by SHA-1 prefix and returns its type and content.

#### Parameters

- `sha1_prefix` (str): Partial or full SHA-1 hash prefix.

#### Returns

- Tuple `(object_type, data_bytes)`.

#### Operation Details

- Locates the full object path in `.git/objects`.
- Reads and decompresses the object.
- Parses header for type and size.
- Returns the object type and data.

#### Example Usage

```python
obj_type, data = read_object('abc123')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### 11. `find_object(sha1_prefix)`

#### Purpose

Finds the full path of an object in the Git object store given a SHA-1 prefix.

#### Parameters

- `sha1_prefix` (str): Partial SHA-1 hash (at least 2 characters).

#### Returns

- Path to the object file in `.git/objects`.

#### Operation Details

- Checks `.git/objects/` directory matching the prefix.
- Ensures exactly one match, otherwise raises error.

#### Example Usage

```python
path = find_object('abc123')
print(f"Object file path: {path}")
```

---

### 12. `write_file(path, data)`

#### Purpose

Writes bytes data to a file at the specified path.

#### Parameters

- `path` (str): File path.
- `data` (bytes): Data to write.

#### Operation Details

- Opens file in binary write mode.
- Writes data to file.

#### Example Usage

```python
write_file('.git/refs/heads/master', b'abc123\n')
```

---

## ASCII Diagram: Git Index Structure and Workflow

```
+-----------------+       +---------------------+       +------------------+
| Working Directory| <---> |       Git Index     | <---> |  Git Object Store |
+-----------------+       +---------------------+       +------------------+
        ^                          ^      ^
        |                          |      |
        |       add(paths)         |      |  hash_object(data, type)
        |------------------------->|      |
        |                          |      |
        |            diff()        |      |
        |<-------------------------|      |
        |                          |      |
        |         write_index()    |      |
        |<-------------------------|      |
        |                          |      |
        |        read_index()      |      |
        |------------------------->|      |
```

- Files in the working directory are added to the index via `add()`.
- The index maintains metadata and SHA-1 hashes.
- `diff()` compares working directory files with staged versions in the index.
- Objects are stored compressed in the Git object store and referenced by SHA-1.

---

## Summary

This documentation covers the essential functions to manipulate the Git index, including reading and writing the index file, managing `IndexEntry` objects, and integrating file changes into Git's object database. These routines are foundational to Git's staging and commit mechanisms, enabling efficient tracking and versioning of file changes.