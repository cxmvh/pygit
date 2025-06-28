# file_io.md

# File I/O and Git Object Operations

## Overview

The **file_io.md** document provides detailed technical documentation on functions responsible for reading and writing files and Git objects within the pygit project. These functions form a foundational layer in pygit for interacting with the filesystem and Git’s internal object storage. This file fits within the broader **Git Index and Working Copy** section of the documentation tree, specifically under the **File Operations** subsection, and supports higher-level operations like diffing, committing, and pushing.

Understanding these functions is critical for grasping how pygit manages data persistence, object integrity, and object retrieval from the `.git` directory. They enable low-level manipulation of files and compressed objects required for Git operations.

---

## Function: read_file

### Purpose

Reads the contents of a file from the filesystem as raw bytes.

### Parameters

- `path` (str): The file path to read.

### Operation

- Opens the specified file in binary read mode.
- Reads all bytes from the file.
- Returns the byte content.

### Usage Example

```python
data = read_file('README.md')
print(data)  # Outputs raw bytes of README.md file
```

---

## Function: write_file

### Purpose

Writes raw bytes to a file on the filesystem.

### Parameters

- `path` (str): The file path to write to.
- `data` (bytes): The raw bytes to write.

### Operation

- Opens the specified file in binary write mode.
- Writes the given bytes to the file.

### Usage Example

```python
write_file('data.bin', b'\x00\x01\x02\x03')
```

---

## Function: find_object

### Purpose

Locates a Git object file in `.git/objects` by SHA-1 prefix.

### Parameters

- `sha1_prefix` (str): The hex SHA-1 prefix (at least 2 characters).

### Operation

- Validates prefix length (minimum 2 chars).
- Uses first two characters as directory name.
- Lists files in this directory matching the remaining prefix.
- Returns the full path to the unique matching object file.
- Raises `ValueError` if no or multiple matches found.

### Usage Example

```python
obj_path = find_object('f4c3b')
print(obj_path)  # e.g., '.git/objects/f4/c3b...'
```

---

## Function: read_object

### Purpose

Reads and parses a Git object by SHA-1 prefix.

### Parameters

- `sha1_prefix` (str): SHA-1 prefix of the object.

### Operation

- Finds object file path using `find_object`.
- Reads and decompresses object data.
- Parses header to get object type and size.
- Extracts object data bytes.
- Validates size consistency.
- Returns tuple `(obj_type, data_bytes)`.

### Usage Example

```python
obj_type, data = read_object('f4c3b2a...')
print(f'Object type: {obj_type}, data length: {len(data)} bytes')
```

---

## Function: hash_object

### Purpose

Hashes and optionally writes object data to the Git object store.

### Parameters

- `data` (bytes): Raw object data.
- `obj_type` (str): Type of object ('blob', 'tree', 'commit', etc.).
- `write` (bool): If True, stores the object in `.git/objects`.

### Operation

- Creates object header: `"type size"`.
- Concatenates header, null byte, and data.
- Computes SHA-1 hash of full data.
- If `write` is True and object not already stored:
  - Compresses and writes object to `.git/objects/xx/yyyy...` path.
- Returns SHA-1 hex string of the object.

### Usage Example

```python
sha1 = hash_object(b'hello world\n', 'blob')
print(f'Object hash: {sha1}')
```

---

## Function: read_index

### Purpose

Reads the Git index file and returns a list of index entries.

### Parameters

None

### Operation

- Reads `.git/index` file bytes.
- Verifies checksum and header signature.
- Parses number of entries.
- Iterates over entry data, unpacking fixed fields and variable-length path.
- Creates and returns list of `IndexEntry` objects.

### Usage Example

```python
index_entries = read_index()
for entry in index_entries:
    print(entry.path, entry.sha1.hex())
```

---

## Function: write_index

### Purpose

Writes a list of index entries to the Git index file.

### Parameters

- `entries` (list of IndexEntry): Entries to write.

### Operation

- Packs each entry's fields and path into binary form.
- Pads entries to 8-byte alignment.
- Builds index header with signature, version, and count.
- Concatenates all entries and computes SHA-1 checksum.
- Writes complete data + checksum to `.git/index`.

### Usage Example

```python
write_index(index_entries)
```

---

## Function: add

### Purpose

Add file paths to the Git index.

### Parameters

- `paths` (list of str): List of file paths to add.

### Operation

- Reads current index entries.
- Removes entries for specified paths.
- For each path:
  - Reads file content.
  - Hashes content as 'blob' object.
  - Creates new index entry with file metadata and SHA-1.
- Sorts entries by path.
- Writes updated index.

### Usage Example

```python
add(['README.md', 'setup.py'])
```

---

## Function: read_tree

### Purpose

Parses a Git tree object into a list of entries.

### Parameters

- `sha1` (str, optional): SHA-1 of tree object to read.
- `data` (bytes, optional): Raw tree data bytes.

### Operation

- If `sha1` provided, reads tree object data.
- Iterates over raw data, splitting entries by null bytes.
- Extracts mode, path, and SHA-1 digest from each entry.
- Returns list of tuples `(mode, path, sha1)`.

### Usage Example

```python
entries = read_tree(sha1='f4c3b2a...')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

## Function: write_tree

### Purpose

Creates a tree object from current index entries and writes it to the object store.

### Parameters

None

### Operation

- Reads index entries.
- Assembles tree entries in format: `"{mode:o} {path}\0{sha1_bytes}"`.
- Joins all entries and hashes as 'tree' object.
- Returns SHA-1 of written tree object.

### Usage Example

```python
tree_sha1 = write_tree()
print(f'Tree object created: {tree_sha1}')
```

---

## Function: cat_file

### Purpose

Outputs content or information about a Git object to stdout.

### Parameters

- `mode` (str): One of `'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, `'pretty'`.
- `sha1_prefix` (str): SHA-1 prefix of object.

### Operation

- Reads object.
- Depending on mode:
  - `'commit'`, `'tree'`, `'blob'`: output raw data.
  - `'size'`: output size of data.
  - `'type'`: output object type.
  - `'pretty'`: pretty-print the object:
    - For `'commit'` and `'blob'`, raw output.
    - For `'tree'`, lists entries with mode, type (tree/blob), SHA-1, and path.

### Usage Example

```python
cat_file('pretty', 'f4c3b2a')
```

---

## ASCII Diagram: Git Object Storage Layout

```
.git/
└── objects/
    ├── ab/
    │   └── cd1234...  # Object file for SHA1 starting with 'abcd1234...'
    ├── f4/
    │   └── c3b2a1...
    └── ...
```

- Objects are stored in directories named after the first two characters of their SHA-1.
- The remaining characters form the filename.
- Objects are compressed and contain a header (type + size) followed by data.

---

## Summary

The functions documented in **file_io.md** enable pygit to:

- Read and write plain files.
- Locate and decompress Git objects using SHA-1 keys.
- Hash and store new Git objects.
- Manage the Git index file and its entries.
- Read and write Git tree objects.
- Display object contents or metadata to the user.

Together, they provide the critical infrastructure for higher-level Git operations such as status checking, diffing, committing, and pushing changes.

---

# End of file_io.md documentation.