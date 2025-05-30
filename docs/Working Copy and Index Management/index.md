# Index File Operations Documentation

## Overview

This document provides comprehensive technical reference for operations on the Git index file (`.git/index`). The index is a core component in Git's architecture, acting as a staging area that holds metadata about files to be committed. This file details how to read and write the index, manipulate its entries, and integrate these operations with status reporting and index management workflows. It is situated within the broader "Working Copy and Index Management" section of the documentation, bridging low-level index file handling with user-facing commands such as `ls-files`, `status`, `diff`, and `commit`.

The index file operations covered here underpin many fundamental Git features, including tracking file changes, staging content, and preparing commit objects. Understanding these operations is essential for anyone implementing or extending Git functionality or building Git-compatible tools.

---

## Function Documentation

---

### `read_index()`

**Purpose:**  
Reads the `.git/index` file and returns a list of `IndexEntry` objects representing the staged files and their metadata.

**Parameters:**  
None

**Returns:**  
`List[IndexEntry]` - A list of index entries with fields such as file creation/modification times, device and inode info, mode, user/group IDs, file size, SHA-1 hash of the content, flags, and file path.

**Operation Details:**  
1. Attempts to open and read the `.git/index` file. Returns an empty list if the file does not exist.  
2. Verifies the file's SHA-1 checksum to ensure integrity.  
3. Parses the index header, checking the signature `DIRC` and version (expected version 2).  
4. Iteratively parses each index entry, unpacking fixed-length fields and the variable-length path name, respecting padding to align entries to an 8-byte boundary.  
5. Assembles and returns a list of `IndexEntry` instances.

**Example Usage:**

```python
entries = read_index()
for entry in entries:
    print(f"{entry.mode:o} {entry.sha1.hex()} {entry.path}")
```

---

### `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the `.git/index` file, updating the index with new or modified entries.

**Parameters:**  
- `entries: List[IndexEntry]` - The index entries to write, usually sorted by path.

**Returns:**  
None

**Operation Details:**  
1. Serializes each index entry into a binary format, packing the fixed-size fields and appending the path with null padding to align to an 8-byte boundary.  
2. Constructs the index file header with signature `DIRC`, version 2, and number of entries.  
3. Computes a SHA-1 checksum of all preceding data for integrity.  
4. Writes the complete binary data plus checksum to `.git/index`.

**Example Usage:**

```python
# Assuming 'entries' is a list of IndexEntry objects
write_index(entries)
print("Index updated with", len(entries), "entries.")
```

---

### `add(paths)`

**Purpose:**  
Adds one or more file paths to the Git index, staging their current content and metadata.

**Parameters:**  
- `paths: List[str]` - List of file paths to add to the index.

**Returns:**  
None

**Operation Details:**  
1. Normalizes paths to use forward slashes.  
2. Reads the current index entries, excluding any that match the added paths (to avoid duplicates).  
3. For each path:  
   - Reads the file content and computes the SHA-1 hash (blob object).  
   - Gathers the filesystem metadata (timestamps, device, inode, mode, UID/GID, size).  
   - Creates a new `IndexEntry` with the above data and appropriate flags.  
4. Appends new entries to the list, sorts entries by path, and writes the updated index.

**Example Usage:**

```python
# Stage 'README.md' and 'src/main.py'
add(['README.md', 'src/main.py'])
print("Files added to index.")
```

---

### `ls_files(details=False)`

**Purpose:**  
Lists files currently staged in the index. Can optionally show detailed information for each entry.

**Parameters:**  
- `details: bool` (default `False`) - If `True`, prints mode, SHA-1, stage number, and path; otherwise, just the path.

**Returns:**  
None (prints output to stdout)

**Operation Details:**  
1. Reads the index entries.  
2. Iterates over entries and prints either the file path alone or detailed info including mode (octal), SHA-1 hash, stage (from flags), and path.

**Example Usage:**

```python
# List files with details
ls_files(details=True)

# Output:
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
# 100644 f572d396fae9206628714fb2ce00f72e94f2258f 0    src/main.py
```

---

### `get_status()`

**Purpose:**  
Determines the working copy status by comparing the file system with the index entries.

**Parameters:**  
None

**Returns:**  
Tuple of three lists:  
- `changed_paths`: Files modified since last indexing.  
- `new_paths`: Files present in working directory but not in the index.  
- `deleted_paths`: Files present in the index but missing from the working directory.

**Operation Details:**  
1. Recursively walks the working directory, excluding `.git`, collecting all file paths.  
2. Reads the current index entries and maps them by path.  
3. Compares the file hashes:  
   - Marks files as changed if the working file content hash differs from the staged hash.  
   - Marks new files if present in working directory but not in index.  
   - Marks deleted files if present in index but not in working directory.

**Example Usage:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `status()`

**Purpose:**  
Prints a human-readable summary of the working copy status.

**Parameters:**  
None

**Returns:**  
None (prints output to stdout)

**Operation Details:**  
1. Calls `get_status()` to obtain changed, new, and deleted files.  
2. Prints categorized lists under headers: "changed files:", "new files:", and "deleted files:" if any exist.

**Example Usage:**

```python
status()

# Output:
# changed files:
#    src/main.py
# new files:
#    docs/README.md
# deleted files:
#    old_script.py
```

---

### `diff()`

**Purpose:**  
Displays unified diffs for files changed between the index and the working directory.

**Parameters:**  
None

**Returns:**  
None (prints diff output to stdout)

**Operation Details:**  
1. Obtains the list of changed files via `get_status()`.  
2. For each changed file:  
   - Reads the staged blob content from the index.  
   - Reads the current working copy file content.  
   - Uses Python's `difflib.unified_diff` to generate a unified diff format.  
3. Prints the diff with filenames annotated as `(index)` and `(working copy)`.  
4. Separates diffs for multiple files with a line of dashes.

**Example Usage:**

```python
diff()

# Output example:
# --- src/main.py (index)
# +++ src/main.py (working copy)
# @@ -1,4 +1,5 @@
#  #!/usr/bin/env python3
# -print("Hello, world!")
# +print("Hello, Git!")
# +
# -print("Done")
# +print("All done")
```

---

### `write_tree()`

**Purpose:**  
Creates a Git tree object from the current index entries and writes it to the object store.

**Parameters:**  
None

**Returns:**  
`str` - SHA-1 hash of the created tree object.

**Operation Details:**  
1. Reads the index entries.  
2. Asserts that paths are all top-level (no directories supported currently).  
3. For each entry, constructs a tree entry of the form `<mode> <path>\0<sha1>`.  
4. Concatenates all tree entries and hashes the combined data as a `tree` object.  
5. Writes the tree object to the object store.  
6. Returns the SHA-1 hash.

**Example Usage:**

```python
tree_hash = write_tree()
print("Tree object created:", tree_hash)
```

---

### `commit(message, author=None)`

**Purpose:**  
Commits the current index state to the local `master` branch with a commit message.

**Parameters:**  
- `message: str` - Commit message.  
- `author: Optional[str]` - Author string in format `"Name <email>"`. If omitted, uses environment variables.

**Returns:**  
`str` - SHA-1 hash of the created commit object.

**Operation Details:**  
1. Calls `write_tree()` to create a tree object from the index.  
2. Retrieves the current `master` branch commit (parent) hash.  
3. Constructs author and committer metadata including timestamp and timezone offset.  
4. Assembles commit content with tree, parent(s), author, committer, and message lines.  
5. Hashes the commit content as a `commit` object and writes it.  
6. Updates the `refs/heads/master` file with the new commit hash.  
7. Prints confirmation and returns the commit hash.

**Example Usage:**

```python
commit_hash = commit("Initial commit")
print("Committed as", commit_hash)
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Locates a Git object file in the object store by its SHA-1 prefix.

**Parameters:**  
- `sha1_prefix: str` - Prefix of the SHA-1 hash (at least 2 characters).

**Returns:**  
`str` - Filesystem path to the object file.

**Raises:**  
- `ValueError` if prefix is shorter than 2 characters, no object matches, or multiple objects match.

**Operation Details:**  
1. Checks prefix length.  
2. Lists files in the `.git/objects/<first two chars>/` directory matching the remaining prefix.  
3. Ensures exactly one match is found.  
4. Returns the full path to the object file.

**Example Usage:**

```python
obj_path = find_object('e69de29')
print("Object file path:", obj_path)
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Reads and decompresses a Git object by its SHA-1 prefix, returning the object type and data.

**Parameters:**  
- `sha1_prefix: str` - SHA-1 prefix identifying the object.

**Returns:**  
Tuple `(obj_type: str, data: bytes)` - The object type (e.g., `commit`, `tree`, `blob`) and raw content bytes.

**Raises:**  
- `ValueError` if the object is not found or corrupted.

**Operation Details:**  
1. Uses `find_object` to locate the object file.  
2. Reads and decompresses the object file using zlib.  
3. Parses the header (e.g., `blob 14\0`) to separate type and size.  
4. Returns the object type and data.

**Example Usage:**

```python
obj_type, data = read_object('e69de29')
print(f"Object type: {obj_type}, size: {len(data)} bytes")
```

---

### `write_file(path, data)`

**Purpose:**  
Writes raw bytes to a given file path.

**Parameters:**  
- `path: str` - File path to write to.  
- `data: bytes` - Data to write.

**Returns:**  
None

**Operation Details:**  
1. Opens the file in binary write mode.  
2. Writes the data bytes.  
3. Closes the file.

**Example Usage:**

```python
write_file('.git/HEAD', b'ref: refs/heads/master\n')
```

---

## Additional Concepts

### Index Entry Structure

Each index entry includes metadata fields such as ctime, mtime, device, inode, mode, UID, GID, size, SHA-1 hash of the blob, flags, and the file path.

```
+------------------+--------------------+
| Field            | Description        |
+------------------+--------------------+
| ctime_s, ctime_n  | Creation time      |
| mtime_s, mtime_n  | Modification time  |
| dev              | Device number      |
| ino              | Inode number       |
| mode             | File mode          |
| uid, gid         | User and Group IDs |
| size             | File size in bytes |
| sha1             | SHA-1 hash of blob |
| flags            | Flags (e.g., stage)|
| path             | File path string   |
+------------------+--------------------+
```

---

## ASCII Diagram: Index File Layout

```
+-----------------------------------------------+
| Header                                        |
| +-------------------------------------------+ |
| | Signature: 'DIRC' (4 bytes)                | |
| | Version: 2 (4 bytes)                       | |
| | Number of entries (4 bytes)                | |
| +-------------------------------------------+ |
+-----------------------------------------------+
| Entry 1                                       |
| +-------------------------------------------+ |
| | Fixed-length fields (ctime, mtime, ... )  | |
| | SHA-1 hash (20 bytes)                       | |
| | Flags (2 bytes)                             | |
| | Path (variable length, null-terminated)    | |
| | Padding to multiple of 8 bytes              | |
| +-------------------------------------------+ |
+-----------------------------------------------+
| Entry 2                                       |
|  ...                                          |
+-----------------------------------------------+
| SHA-1 checksum of all previous bytes (20)    |
+-----------------------------------------------+
```

---

## Index Management Flow (Simplified)

```
Working Directory
       |
       | (1) read_file(path)
       v
     add(paths) ------------------> read_index()
       |                               |
       | (2) hash_object(blob)          |
       |                               v
       |                      modify entries list
       |                               |
       |-----------------------------> write_index(entries)
       |
       v
    ls_files(details)
       |
       v
    status() ---> get_status() --> compare working directory and index
       |
       v
    diff() ---> read_object(sha1) --> show diffs
       |
       v
    commit(message) --> write_tree() --> write commit object --> update refs
```

---

This documentation is intended to serve as a detailed guide for developers working with Git's index file operations within the pygit implementation, providing both conceptual understanding and practical code usage examples.