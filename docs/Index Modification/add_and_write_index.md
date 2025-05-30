# add_and_write_index.md

# Documentation for Adding Files to Index and Writing the Index File

---

## Overview

This document covers the functionality involved in adding files to the Git index (also known as the staging area) and writing the updated index file to disk. The Git index is a critical data structure that tracks the state of files staged for the next commit. Proper manipulation of the index is essential for accurate version control operations.

Within the broader documentation hierarchy, this file is part of the **Index Modification** section, which focuses on operations that modify the index, including adding files and updating the index file. This documentation complements related topics such as index reading (`index.md`), status reporting (`status.md`), and core Git commands.

The primary functions documented here include `add(paths)`, which stages files by adding them to the index, and `write_index(entries)`, which serializes and writes the index entries to the `.git/index` file. Understanding these functions facilitates comprehension of how Git tracks changes and prepares them for commits.

---

## Function Documentation

### `add(paths)`

#### Purpose
Add one or more files to the Git index, staging them for the next commit.

#### Parameters
- `paths` (list of str): A list of filesystem paths of files to be added to the index.

#### Preconditions
- The files specified in `paths` must exist in the working directory.
- The `.git` directory must be properly initialized.

#### Description

The `add` function stages files by performing the following steps:

1. Normalize the paths to use forward slashes (`/`) for compatibility.
2. Read the current index entries using `read_index()`.
3. Filter out any existing index entries for the files being added, to avoid duplication.
4. For each file path:
   - Read the file contents from disk.
   - Compute the SHA-1 hash of the file content using `hash_object(data, 'blob')`. This stores the file as a Git blob object in the object database.
   - Retrieve file metadata (timestamps, device, inode, mode, ownership, size) using `os.stat`.
   - Calculate flags based on the length of the file path (must fit in 12 bits).
   - Create a new `IndexEntry` object representing the file's index entry with collected metadata and SHA-1.
5. Append the new entries to the filtered list.
6. Sort all entries lexicographically by their file paths.
7. Write the updated list of index entries back to the `.git/index` file using `write_index(entries)`.

#### Detailed Operation Flow

```
+------------------+
| Input file paths  |
+---------+--------+
          |
          v
+-------------------------+
| Normalize path format   |
+-------------------------+
          |
          v
+---------------------+
| Read current index   |
+---------------------+
          |
          v
+-----------------------------+
| Remove entries for new paths |
+-----------------------------+
          |
          v
+-----------------------------------------------+
| For each new path:                             |
| - Read file content                           |
| - Hash content as 'blob'                      |
| - Get file stat info                          |
| - Create IndexEntry with metadata and SHA-1  |
+-----------------------------------------------+
          |
          v
+--------------------------+
| Append new entries       |
+--------------------------+
          |
          v
+----------------------------+
| Sort entries by file path   |
+----------------------------+
          |
          v
+------------------------------+
| Write updated index to disk   |
+------------------------------+
```

#### Example Usage

```python
# Stage a single file
add(['README.md'])

# Stage multiple files
add(['src/main.py', 'tests/test_main.py'])
```

---

### `write_index(entries)`

#### Purpose
Serialize and write a list of `IndexEntry` objects to the `.git/index` file in the correct Git index file format.

#### Parameters
- `entries` (list of `IndexEntry`): A list of index entry objects representing the files staged.

#### Preconditions
- The `entries` list should be sorted by file path for consistency.
- Each `IndexEntry` must have valid metadata and SHA-1 hash.

#### Description

The `write_index` function performs the following:

1. For each `IndexEntry`:
   - Pack the fixed-size metadata fields (timestamps, device, inode, mode, uid, gid, size, SHA-1, flags) into a binary structure using `struct.pack`.
   - Encode the file path as bytes.
   - Calculate the total length of the entry, padded to a multiple of 8 bytes.
   - Append null bytes as padding to align the entry size.
2. Prepend the index header consisting of:
   - Signature: ASCII string `"DIRC"`.
   - Version number (currently 2).
   - Number of entries.
3. Concatenate the packed header and all packed entries.
4. Compute a SHA-1 checksum of the entire index data (excluding the checksum itself).
5. Append the checksum to the end of the data.
6. Write the complete data to `.git/index` using `write_file`.

#### Detailed Operation Flow

```
+-----------------------------+
| For each IndexEntry:         |
|  Pack metadata fields        |
|  Encode and pad file path    |
+-----------------------------+
           |
           v
+-----------------------------+
| Build index header:          |
|  - Signature "DIRC"          |
|  - Version (2)              |
|  - Number of entries         |
+-----------------------------+
           |
           v
+-----------------------------+
| Concatenate header + entries|
+-----------------------------+
           |
           v
+-----------------------------+
| Compute SHA-1 checksum      |
+-----------------------------+
           |
           v
+-----------------------------+
| Append checksum to data     |
+-----------------------------+
           |
           v
+-----------------------------+
| Write data to .git/index    |
+-----------------------------+
```

#### Example Usage

```python
# Assume 'entries' is a list of IndexEntry objects
write_index(entries)
```

---

## Additional Context and Related Functions

- `read_index()`: Reads the current `.git/index` file and returns a list of `IndexEntry` objects.
- `hash_object(data, obj_type)`: Hashes object data and writes it to the Git object store if needed.
- `write_file(path, data)`: Writes raw bytes to a file.
- `IndexEntry`: Data structure representing an index entry, holding file metadata and object hash.

---

## Summary Diagram: Adding Files to Index and Writing Index

```
Working Directory Files
        |
        v
   add(paths)
        |
        v
Computes SHA-1 hash of file blobs
        |
        v
Creates/Updates IndexEntry objects
        |
        v
write_index(entries)
        |
        v
Writes .git/index file with updated entries
```

---

This documentation provides a detailed understanding of how files are staged in Git via the index and how the index file is serialized and stored. Mastery of these operations is crucial for implementing or extending Git-like version control functionality.