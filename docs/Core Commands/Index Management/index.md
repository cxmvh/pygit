# index.md - Git Index File Handling Documentation

---

## Overview

This document provides comprehensive technical reference for reading, writing, and manipulating the Git index file, a critical component in the Git version control system. The index file, often called the staging area, holds a snapshot of the working directory that is prepared to be committed. This documentation details the `IndexEntry` structure representation, the procedures to read and write the index file, and functions to add files to the index. It fits within the broader "Core Commands" section under "Index Management" and relates to repository initialization, committing, and status reporting workflows.

---

## Function Documentation

### IndexEntry Structure

The `IndexEntry` is a data structure representing a single entry in the Git index. Each entry corresponds to a file tracked by Git and contains metadata and the SHA-1 hash of the file's blob object.

- **Fields include:**
  - `ctime_s`, `ctime_n`: File creation time (seconds and nanoseconds).
  - `mtime_s`, `mtime_n`: File modification time (seconds and nanoseconds).
  - `dev`, `ino`: Device and inode number.
  - `mode`: File mode (permissions).
  - `uid`, `gid`: User and group IDs.
  - `size`: File size in bytes.
  - `sha1`: SHA-1 hash of the blob object.
  - `flags`: Flags including path length.
  - `path`: File path (relative to repository root).

---

### read_index()

**Purpose:**  
Reads the Git index file from `.git/index` and parses it into a list of `IndexEntry` objects.

**Parameters:**  
None.

**Returns:**  
List of `IndexEntry` instances representing the current state of the index.

**Operation:**  
1. Reads the raw index file bytes.
2. Verifies the SHA-1 checksum at the end of the file to ensure integrity.
3. Parses the header to confirm format signature (`DIRC`) and version (must be 2).
4. Iteratively unpacks each fixed-size entry header and extracts the file path.
5. Handles padding to align entries to an 8-byte boundary.
6. Returns the list of parsed entries.

**Example:**

```python
entries = read_index()
for entry in entries:
    print(f"File: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### write_index(entries)

**Purpose:**  
Writes a list of `IndexEntry` objects back to the Git index file, updating the staging area.

**Parameters:**  
- `entries` (List[IndexEntry]): Sorted list of index entries to write.

**Returns:**  
None.

**Operation:**  
1. Packs each `IndexEntry` into its binary representation, including header fields and path.
2. Pads each entry to an 8-byte boundary.
3. Concatenates all entries with a `DIRC` file header (signature, version, number of entries).
4. Computes and appends a SHA-1 checksum of all preceding data.
5. Writes the final data blob to `.git/index`.

**Example:**

```python
entries = read_index()
# Modify entries as needed
write_index(entries)
```

---

### add(paths)

**Purpose:**  
Adds one or more file paths to the Git index, preparing them for commit.

**Parameters:**  
- `paths` (List[str]): List of file paths relative to the repository root.

**Returns:**  
None.

**Operation:**  
1. Normalizes paths to use forward slashes.
2. Reads the current index entries.
3. Removes any existing entries matching the paths to be added.
4. For each new path:
   - Reads file content and computes its SHA-1 hash (blob object).
   - Retrieves file metadata (stat).
   - Creates a new `IndexEntry` with metadata and the computed hash.
5. Combines existing and new entries, sorts them by path.
6. Writes the updated entries back to the index.

**Example:**

```python
add(['src/main.py', 'README.md'])
```

---

### read_file(path)

**Purpose:**  
Utility function to read file contents as bytes.

**Parameters:**  
- `path` (str): Path to the file.

**Returns:**  
Bytes read from the file.

**Example:**

```python
data = read_file('README.md')
print(data.decode())
```

---

### write_file(path, data)

**Purpose:**  
Utility function to write byte data to a file.

**Parameters:**  
- `path` (str): Destination file path.
- `data` (bytes): Data to write.

**Returns:**  
None.

**Example:**

```python
write_file('.git/index', index_data)
```

---

### hash_object(data, obj_type, write=True)

**Purpose:**  
Computes the SHA-1 hash of an object of a given type and optionally writes it to the object store.

**Parameters:**  
- `data` (bytes): Object content data.
- `obj_type` (str): Object type, e.g., `'blob'`, `'tree'`, `'commit'`.
- `write` (bool): Whether to write the compressed object to `.git/objects`.

**Returns:**  
SHA-1 hash string of the object.

**Example:**

```python
sha1 = hash_object(b'Hello, Git!', 'blob')
print(f"Object SHA-1: {sha1}")
```

---

### write_tree()

**Purpose:**  
Creates a Git tree object from the current index entries representing files staged at the top-level directory.

**Parameters:**  
None.

**Returns:**  
SHA-1 hash of the created tree object.

**Operation:**  
1. Reads current index entries.
2. Asserts all entries are top-level (no slashes in path).
3. For each entry, encodes mode and path, concatenates with SHA-1 of blob.
4. Joins all entries to form tree content.
5. Hashes and writes the tree object using `hash_object`.

**Example:**

```python
tree_sha1 = write_tree()
print(f"Tree object SHA-1: {tree_sha1}")
```

---

### get_status()

**Purpose:**  
Compares the working directory files against the index to determine changed, new, and deleted files.

**Parameters:**  
None.

**Returns:**  
Tuple of three lists: `(changed_paths, new_paths, deleted_paths)`.

**Operation:**  
1. Walks the working directory (excluding `.git`).
2. Collects all file paths.
3. Reads the index entries and maps by path.
4. Determines:
   - Changed files: Present in both but with differing SHA-1 hashes.
   - New files: Present in working directory but not in index.
   - Deleted files: Present in index but missing in working directory.

**Example:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### status()

**Purpose:**  
Prints the working copy status to stdout, showing changed, new, and deleted files.

**Parameters:**  
None.

**Returns:**  
None.

**Example:**

```python
status()
```

**Output:**

```
changed files:
    src/main.py
new files:
    docs/manual.md
deleted files:
    old_script.py
```

---

### diff()

**Purpose:**  
Displays unified diffs between the index and the working copy for changed files.

**Parameters:**  
None.

**Returns:**  
None.

**Operation:**  
1. Gets changed file paths.
2. For each changed file:
   - Reads blob content from index.
   - Reads current working file content.
   - Uses `difflib.unified_diff` to generate and print diff.

**Example:**

```python
diff()
```

---

### ls_files(details=False)

**Purpose:**  
Lists files currently in the index.

**Parameters:**  
- `details` (bool): If `True`, includes mode, SHA-1, and stage number.

**Returns:**  
None; prints output to stdout.

**Example:**

```python
ls_files(details=True)
```

**Sample output:**

```
100644 3b18e6f7e8aabcd1234abcd5678ef90123456789 0    README.md
100755 9c8fe5a7b8dabcde56789abcd1234567890abcdef 0    script.sh
```

---

## ASCII Diagram: Git Index and File Workflow

```
Working Directory
   |
   |  (git add)
   v
+------------------+
|   Git Index      | <---- (read_index, write_index)
+------------------+
   |
   |  (git commit)
   v
+------------------+
|   Git Tree &     |
|   Commit Objects |
+------------------+
```

- Files in the working directory are added to the index via `add(paths)`.
- The index stores metadata and blob hashes of files.
- Committing converts index entries into tree and commit objects.

---

## Summary

The Git index file is a binary file that maintains metadata and blob references for files staged for commit. This document covers reading (`read_index`), writing (`write_index`), and updating the index (`add`) with detailed explanations and example code snippets. Additionally, utility functions for file I/O and hashing, as well as status and diff commands interacting with the index, are documented to provide a complete view of index management in the pygit project.