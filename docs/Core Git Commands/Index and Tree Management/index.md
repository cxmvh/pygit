# index.md

# Reading and Writing the Git Index

---

## Overview

This document details how the Git index is read from and written to within the pygit repository implementation. The Git index is a critical data structure that acts as a staging area between the working directory and the repository’s commit history. It keeps track of the files and their states that are ready to be committed. This file (`index.md`) fits within the broader "Index and Working Copy Management" section of the documentation tree, which collectively covers index file manipulation, status reporting, diffs, and file operations. Understanding how to read and write the index is fundamental for implementing core Git commands such as `add`, `commit`, and `status`.

---

## Function Documentation

### read_index()

**Purpose:**  
Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing the staged files.

**Parameters:**  
None

**Returns:**  
`List[IndexEntry]` — List of index entries parsed from the index file.

**Operation:**  
1. Attempts to read the `.git/index` file as binary data. If the file does not exist, returns an empty list indicating no index entries.  
2. Validates the integrity of the index by verifying the SHA-1 checksum of the file contents (excluding the last 20 bytes, which store the checksum).  
3. Parses the index header to confirm the signature (`DIRC`) and version (expected `2`).  
4. Iteratively parses each index entry consisting of metadata fields, SHA-1 blob, flags, and the file path. The entry size is padded to align to an 8-byte boundary.  
5. Returns a fully populated list of `IndexEntry` objects.

**Example Usage:**
```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}")
```

---

### write_index(entries)

**Purpose:**  
Writes a list of `IndexEntry` objects to the Git index file, updating the staging area.

**Parameters:**  
- `entries` (`List[IndexEntry]`): The list of index entries to write.

**Returns:**  
None

**Operation:**  
1. Packs each `IndexEntry` into the binary format expected by the Git index file, including all metadata fields, SHA-1, flags, and path.  
2. Ensures padding of each entry to an 8-byte boundary for alignment.  
3. Constructs the index file header with signature `DIRC`, version `2`, and the number of entries.  
4. Concatenates the header and packed entries, then computes and appends the SHA-1 checksum of the entire content.  
5. Writes the resulting binary data to `.git/index`.

**Example Usage:**
```python
from pygit import read_index, write_index

entries = read_index()
# Modify entries as needed, e.g., add or remove files
write_index(entries)
```

---

### add(paths)

**Purpose:**  
Adds one or more file paths to the Git index by hashing their contents and updating the index entries.

**Parameters:**  
- `paths` (`List[str]`): List of file paths to add to the index.

**Returns:**  
None

**Operation:**  
1. Normalizes the file paths to use forward slashes for consistency.  
2. Reads the existing index entries and filters out any entries matching the paths being added to avoid duplicates.  
3. For each path:  
    - Reads the file contents and hashes them as a `blob` object, storing the SHA-1.  
    - Gathers file metadata such as timestamps, device, inode, mode, user/group IDs, and size.  
    - Creates a new `IndexEntry` with this data and the file path.  
4. Appends new entries and sorts the entire list by path.  
5. Writes the updated index back to `.git/index`.

**Example Usage:**
```python
add(['src/main.py', 'README.md'])
```

---

### get_status()

**Purpose:**  
Determines the status of the working copy compared to the Git index, identifying changed, new, and deleted files.

**Parameters:**  
None

**Returns:**  
Tuple of three lists (`changed_paths`, `new_paths`, `deleted_paths`)

**Operation:**  
1. Recursively walks the working directory (excluding `.git`) to collect all file paths currently present.  
2. Reads the index entries and builds a mapping from path to entry.  
3. Determines:  
    - **Changed:** Files present both in working directory and index but whose content hashes differ.  
    - **New:** Files present in working directory but not in the index.  
    - **Deleted:** Files present in the index but missing from the working directory.

**Example Usage:**
```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### write_tree()

**Purpose:**  
Creates a Git tree object from the current index entries representing the top-level directory.

**Parameters:**  
None

**Returns:**  
`str` — SHA-1 hash of the newly created tree object.

**Operation:**  
1. Reads the current index entries.  
2. For each entry, asserts that the path contains no directory separator (`/`), meaning only top-level files are supported.  
3. Constructs a tree entry by combining the file mode, path, and SHA-1 as defined by Git tree object format.  
4. Joins all tree entries and hashes them as a `tree` object to store in the object database.  

**Example Usage:**
```python
tree_sha1 = write_tree()
print(f"Created tree object with SHA-1: {tree_sha1}")
```

**ASCII Diagram:**

```
Git Index Entries
+----------------+       +---------------------+
| Mode | Path | SHA1 | --> | Tree Entry Format   |
+----------------+       +---------------------+
         |                           |
         +---------------------------+
                     |
             Concatenate Entries
                     |
             Hash as Tree Object
                     |
             Store in Object DB
```

---

### read_file(path)

**Purpose:**  
Reads the contents of a file as bytes.

**Parameters:**  
- `path` (`str`): Path to the file.

**Returns:**  
`bytes` — Contents of the file.

**Operation:**  
Opens the file in binary mode and reads the contents.

**Example Usage:**
```python
data = read_file('README.md')
print(data.decode())
```

---

### write_file(path, data)

**Purpose:**  
Writes bytes data to a file.

**Parameters:**  
- `path` (`str`): Path of the file to write.  
- `data` (`bytes`): Data to write.

**Returns:**  
None

**Operation:**  
Opens the file in binary write mode and writes the data.

**Example Usage:**
```python
write_file('.git/index', index_data)
```

---

## Summary

This documentation covers the core mechanisms for reading, writing, and manipulating the Git index file within the pygit implementation. The index acts as an in-memory representation of the staged snapshot of the working directory, essential for committing and managing repository state. Functions like `read_index` and `write_index` handle serialization and deserialization of the index file, while `add` updates the index with new files. Coupled with `write_tree`, these functions enable building commit objects representing repository snapshots.

Understanding these components provides a solid foundation for deeper operations such as committing changes, inspecting status, and generating diffs, all of which rely on accurate and efficient index management.