# index_management.md

# Managing Git Index Files and Staging Changes

---

## Overview

This document provides a detailed technical reference for managing Git index files and staging changes within a Git repository. It focuses on the Git index file format—how to read, interpret, and write the index—and describes how to handle index entries representing file states in the working directory. Situated within the broader **Index and Working Copy Management** section of the documentation tree, this file complements other materials on listing files, showing status, diffs, and adding files to the index. The functions documented here underpin core operations such as staging changes before a commit and are integral to workflows executed by commands like `git add` and `git push`.

The documentation covers reading and writing the binary Git index file (`.git/index`), manipulating index entries, and coordinating staged content with the working copy and repository. It also connects to related code flows such as `pygit.push` and `pygit.ls_files`, showing the practical role of index management in pushing commits and listing indexed files.

---

## Function Documentation

---

### `read_index()`

#### Purpose

Reads the Git index file from `.git/index` and parses it into a list of index entries, each represented as an `IndexEntry` object. The index file contains metadata about files staged for commit, including timestamps, file modes, SHA-1 object hashes, and pathnames.

#### Parameters

- None

#### Returns

- `List[IndexEntry]`: List of index entries representing files staged in the index.

#### Preconditions

- The `.git/index` file must exist and be well-formed according to Git's index file format (version 2 supported).
- The index file must pass checksum verification.

#### Operation Details

1. Attempts to read the `.git/index` file as raw bytes.
2. Validates the SHA-1 checksum at the end of the file to ensure data integrity.
3. Unpacks the header:
   - Signature (`b'DIRC'`)
   - Version number (must be 2)
   - Number of entries
4. Iterates over the entries segment:
   - Each entry has a fixed 62-byte header followed by a null-terminated pathname string.
   - Uses `struct.unpack` to extract metadata fields (timestamps, device, inode, mode, user ID, group ID, size, SHA-1 hash, flags).
   - Decodes the pathname from bytes.
   - Aligns entries to an 8-byte boundary.
5. Collects each entry into an `IndexEntry` list.
6. Checks that the number of parsed entries matches the header count.

#### Example Usage

```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}")
```

---

### `write_index(entries)`

#### Purpose

Writes a list of `IndexEntry` objects back to the `.git/index` file, serializing them in the Git index file format (version 2) and appending the SHA-1 checksum.

#### Parameters

- `entries` (`List[IndexEntry]`): List of index entries to write.

#### Operation Details

1. For each `IndexEntry`:
   - Packs metadata fields into a 62-byte header using `struct.pack`.
   - Encodes the pathname as bytes.
   - Pads the entry to the nearest 8-byte boundary with null bytes.
2. Constructs the index file header:
   - Signature `b'DIRC'`
   - Version number 2
   - Number of entries
3. Concatenates header and all packed entries.
4. Computes SHA-1 checksum of all data excluding the checksum itself.
5. Writes the complete data plus checksum to `.git/index`.

#### Example Usage

```python
entries = read_index()
# Modify entries as needed, e.g., add or remove files
write_index(entries)
```

---

### `add(paths)`

#### Purpose

Adds one or more file paths from the working directory to the Git index, staging their current content for commit.

#### Parameters

- `paths` (`List[str]`): List of file paths to add to the index.

#### Operation Details

1. Normalizes paths to use forward slashes.
2. Reads existing index entries.
3. Removes any entries from the current index matching the given paths.
4. For each specified path:
   - Reads file content.
   - Hashes content as a Git blob object, writing it to the object store.
   - Retrieves file system metadata (ctime, mtime, device, inode, mode, uid, gid, size).
   - Computes flags (pathname length, stage number).
   - Constructs a new `IndexEntry`.
5. Adds new entries to the index list.
6. Sorts entries by pathname.
7. Writes updated index back to disk using `write_index`.

#### Example Usage

```python
add(['README.md', 'src/main.py'])
```

---

### `ls_files(details=False)`

#### Purpose

Lists all files recorded in the Git index, optionally displaying additional details such as file mode, SHA-1 hash, and staging information.

#### Parameters

- `details` (`bool`): If `True`, prints detailed info; otherwise, prints only pathnames.

#### Operation Details

1. Reads the index entries.
2. Iterates over each entry and:
   - If `details` is `False`, prints only the file path.
   - If `details` is `True`, extracts and prints mode (octal), SHA-1 hex, stage number (extracted from flags), and path.

#### Example Usage

```python
ls_files(details=True)
```

Output example:

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	README.md
100755 5e3ee119867225f4d3b688d7d1f6f2f9e8266e29 0	src/main.py
```

---

### `get_status()`

#### Purpose

Checks the current state of the working copy compared to the index, returning lists of changed, new, and deleted files.

#### Returns

- Tuple of three lists:
  - `changed_paths`: Files modified in the working directory relative to the index.
  - `new_paths`: Files present in the working copy but not in the index.
  - `deleted_paths`: Files present in the index but missing in the working copy.

#### Operation Details

1. Walks the working directory recursively, ignoring `.git`.
2. Collects normalized paths of all files found.
3. Reads current index entries.
4. Determines:
   - Changed files: filenames present in both index and working directory but with differing blob hashes.
   - New files: present in working directory but not in index.
   - Deleted files: present in index but missing from working directory.

#### Example Usage

```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### `diff()`

#### Purpose

Displays a unified diff between the contents of files in the index and their current state in the working copy, highlighting changes staged for commit.

#### Operation Details

1. Obtains the list of changed files using `get_status`.
2. Reads index entries into a dictionary keyed by path.
3. For each changed file:
   - Reads the blob object from the index.
   - Reads the current working directory file.
   - Uses Python’s `difflib.unified_diff` to generate diff lines.
   - Prints diff output with headers indicating index and working copy versions.

#### Example Usage

```python
diff()
```

Sample diff output:

```
--- README.md (index)
+++ README.md (working copy)
@@ -1 +1,2 @@
-Hello World
+Hello World
+New line added
```

---

### `write_tree()`

#### Purpose

Constructs a Git tree object representing the current state of the index (only supports a single-level directory) and writes it to the object store, returning the tree’s SHA-1 hash.

#### Returns

- SHA-1 hash (hex string) of the created tree object.

#### Operation Details

1. Reads all index entries.
2. Asserts no nested directories (pathname does not contain '/').
3. For each entry:
   - Formats a tree entry as `<mode> <path>\0<raw SHA-1 bytes>`.
4. Concatenates all entries and hashes them as a `tree` object.
5. Writes the tree object to the object store.
6. Returns the SHA-1 hash of the tree.

#### Example Usage

```python
tree_sha1 = write_tree()
print(f"Tree SHA-1: {tree_sha1}")
```

---

### ASCII Diagram: Git Index File Structure

```
+-----------------------+
| Header (12 bytes)      |
|-----------------------|
| Signature: 4 bytes "DIRC"  |
| Version:   4 bytes          |
| NumEntries:4 bytes          |
+-----------------------+
| Entry 1                |
| - Metadata (62 bytes)  |
| - Pathname (variable)  |
| - Padding to 8-byte boundary |
+-----------------------+
| Entry 2                |
| ...                    |
+-----------------------+
| ...                    |
+-----------------------+
| SHA-1 checksum (20 bytes) |
+-----------------------+
```

Each index entry contains file metadata including timestamps, device/inode info, mode, user/group IDs, file size, SHA-1 hash of blob, flags, and the pathname string terminated by a null byte.

---

## Summary

This documentation covers key functions for managing the Git index, a crucial intermediary between the working directory and Git’s object database. Functions to read and write the index file enable staging and tracking file changes for commits. Additional utilities such as `add`, `ls_files`, `get_status`, and `diff` leverage index management to implement typical Git workflows. Understanding these components is essential for anyone working on Git internals or implementing Git-compatible tools.