# object_storage.md

---

## Overview

This document details the handling of Git objects within the pygit repository implementation. It focuses on the processes of hashing, reading, writing, and locating Git objects in the object store, which are fundamental operations for managing the repository's data integrity and history. Positioned under the "Repository Initialization and Object Management" section and specifically within "Object Management," this file complements other documentation on Git objects, such as general object handling (`objects.md`) and tree/commit-specific operations. The functions here are critical for low-level Git object manipulation, enabling higher-level commands like status, commit, and push to function effectively.

---

## Function Documentation

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of the given object data with the specified Git object type (e.g., blob, tree, commit). Optionally writes the compressed object data into the Git object store.

**Parameters:**  
- `data` (bytes): The raw content of the object to be hashed.  
- `obj_type` (str): The Git object type (e.g., `'blob'`, `'tree'`, `'commit'`).  
- `write` (bool, optional): If `True`, writes the object to the object store. Defaults to `True`.

**How it works:**  
1. Constructs a header combining the object type and length, separated by a space, followed by a null byte.  
2. Concatenates the header and the data to form the full object representation.  
3. Computes the SHA-1 hash of the full object data.  
4. If `write` is `True`, compresses and writes the object to a file located under `.git/objects/<first two hash chars>/<remaining hash chars>`. Creates directories as needed.  
5. Returns the SHA-1 hash in hexadecimal form.

**Example Usage:**
```python
data = b"Hello, Git!"
sha1_hash = hash_object(data, 'blob')
print(f"Object stored with SHA-1: {sha1_hash}")
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Reads and decompresses a Git object from the object store using its SHA-1 hash prefix. Returns the object type and the raw data bytes.

**Parameters:**  
- `sha1_prefix` (str): Prefix of the SHA-1 hash identifying the object.

**How it works:**  
1. Uses `find_object()` to locate the full path to the object file corresponding to the SHA-1 prefix.  
2. Reads and decompresses the object file data.  
3. Parses the header to extract the object type and size.  
4. Validates that the size matches the length of the data.  
5. Returns a tuple `(object_type, data_bytes)`.

**Example Usage:**
```python
obj_type, data = read_object('e69de29')  # prefix of SHA-1 for an empty blob
print(f"Object type: {obj_type}")
print(f"Object data: {data}")
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Finds the path to a Git object file in the object store, given a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (str): The prefix of the SHA-1 hash (at least 2 characters).

**How it works:**  
1. Validates that the prefix length is at least 2 characters.  
2. Searches the directory `.git/objects/<first two chars>` for files starting with the remaining prefix.  
3. Raises an error if no objects or multiple objects match the prefix.  
4. Returns the filesystem path to the object.

**Example Usage:**
```python
obj_path = find_object('e69de29')
print(f"Object path: {obj_path}")
```

---

### `write_file(path, data)`

**Purpose:**  
Writes bytes data to a file at the specified path.

**Parameters:**  
- `path` (str): File path to write data to.  
- `data` (bytes): Data to write.

**How it works:**  
- Opens the file in binary write mode and writes the data.

**Example Usage:**
```python
write_file('.git/objects/ab/cdef1234', b'compressed data')
```

---

### `read_file(path)`

**Purpose:**  
Reads the entire content of a file as bytes.

**Parameters:**  
- `path` (str): File path to read from.

**How it works:**  
- Opens the file in binary read mode and returns its contents.

**Example Usage:**
```python
content = read_file('README.md')
print(content.decode())
```

---

### `cat_file(mode, sha1_prefix)`

**Purpose:**  
Prints the contents or metadata of a Git object identified by a SHA-1 prefix to standard output. Supports modes to show raw data, size, type, or a pretty-printed format.

**Parameters:**  
- `mode` (str): One of `'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, `'pretty'`.  
- `sha1_prefix` (str): SHA-1 prefix of the object.

**How it works:**  
1. Reads the object via `read_object()`.  
2. Depending on mode:  
   - For `'commit'`, `'tree'`, `'blob'`: verifies the type and writes raw data bytes to stdout.  
   - For `'size'`: prints the size of the object data.  
   - For `'type'`: prints the Git object type.  
   - For `'pretty'`: prints a formatted view; for trees, prints mode, type, SHA-1, and path per entry.

**Example Usage:**
```bash
$ pygit cat-file pretty e69de29
```

---

### `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the Git index file `.git/index`.

**Parameters:**  
- `entries` (list): List of `IndexEntry` objects representing the index.

**How it works:**  
1. Packs each index entry into its binary form including metadata and path.  
2. Constructs the index header with signature, version, and entry count.  
3. Calculates and appends a SHA-1 checksum for integrity.  
4. Writes the packed data to the index file.

**Example Usage:**
```python
entries = read_index()
write_index(entries)
```

---

### `read_index()`

**Purpose:**  
Reads the Git index file and returns a list of `IndexEntry` objects.

**How it works:**  
1. Reads the `.git/index` file bytes.  
2. Validates signature and version.  
3. Parses entries sequentially, unpacking each field and path.  
4. Verifies checksum.  
5. Returns the list of entries.

**Example Usage:**
```python
entries = read_index()
for e in entries:
    print(f"{e.path}: {e.sha1.hex()}")
```

---

### `add(paths)`

**Purpose:**  
Adds one or more file paths to the Git index, hashing their contents and updating the index entries.

**Parameters:**  
- `paths` (list of str): File paths to add to the index.

**How it works:**  
1. Normalizes paths to POSIX style.  
2. Reads current index entries, excluding those with paths to be added.  
3. For each path:  
   - Reads file contents and hashes as blob object.  
   - Collects file metadata (timestamps, mode, owner).  
   - Creates a new `IndexEntry`.  
4. Sorts entries by path and writes back to the index.

**Example Usage:**
```python
add(['README.md', 'src/main.py'])
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Reads a Git tree object and returns a list of tuples `(mode, path, sha1)` representing its entries.

**Parameters:**  
- `sha1` (str, optional): SHA-1 of the tree object to read.  
- `data` (bytes, optional): Raw tree data; if provided, `sha1` is ignored.

**How it works:**  
1. Reads the tree object data using `read_object()` if SHA-1 is provided.  
2. Iterates over entries by scanning for null-terminated strings separating mode/path and SHA-1.  
3. Parses mode (in octal), path, and the 20-byte SHA-1 digest.  
4. Returns the list of entries.

**Example Usage:**
```python
entries = read_tree('4b825dc642cb6eb9a060e54bf8d69288fbee4904')
for mode, path, sha1 in entries:
    print(f"{oct(mode)} {sha1} {path}")
```

---

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively finds all object SHA-1 hashes referenced by a tree object, including nested trees and blobs.

**Parameters:**  
- `tree_sha1` (str): SHA-1 hash of the tree object.

**How it works:**  
1. Starts with a set containing the tree SHA-1 itself.  
2. Reads the tree entries.  
3. For each entry:  
   - If it is a directory (tree), recursively collects objects from it.  
   - Otherwise, adds the blob SHA-1.  
4. Returns the complete set of object SHA-1 hashes.

**Example Usage:**
```python
objects = find_tree_objects('4b825dc642cb6eb9a060e54bf8d69288fbee4904')
print(objects)
```

---

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively finds all object SHA-1 hashes referenced by a commit, including its tree, parent commits, and all associated objects.

**Parameters:**  
- `commit_sha1` (str): SHA-1 hash of the commit object.

**How it works:**  
1. Adds the commit SHA-1 to the set.  
2. Reads the commit object data.  
3. Extracts the tree SHA-1 and recursively collects tree objects.  
4. Extracts parent commit SHA-1s and recursively collects their objects.  
5. Returns the complete set.

**Example Usage:**
```python
objects = find_commit_objects('d670460b4b4aece5915caf5c68d12f560a9fe3e4')
```

---

### `status()`

**Purpose:**  
Shows the status of the working copy by listing changed, new, and deleted files.

**How it works:**  
1. Calls `get_status()` to get lists of changed, new, and deleted files.  
2. Prints each category with the file paths.

**Example Usage:**
```python
status()
```

---

### `get_status()`

**Purpose:**  
Determines which files in the working directory are changed, new, or deleted relative to the index.

**Returns:**  
Tuple of three lists: `(changed_paths, new_paths, deleted_paths)`.

**How it works:**  
1. Walks the working directory, ignoring `.git`.  
2. Compares file paths against the index entries.  
3. For files present in both, compares blob hashes to detect changes.  
4. Identifies new files (in working dir but not index).  
5. Identifies deleted files (in index but not working dir).

**Example Usage:**
```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### Additional Supporting Functions

- `write_tree()`: Creates a tree object from current index entries and returns its SHA-1.  
- `commit(message, author=None)`: Commits the current index state with a commit message and author info.  
- `ls_files(details=False)`: Lists files in the index, optionally with detailed info (mode, SHA-1, stage).  

---

## Illustrative ASCII Diagram: Git Object Storage Layout

```
.git/
 ├── objects/
 │    ├── ab/
 │    │    └── cdef1234...   # Object file named by SHA-1 suffix
 │    ├── e6/
 │    │    └── 9de29bb2...   # Another object file
 │    └── ...                # More object directories
 ├── index                 # Git index file listing tracked files
 ├── refs/
 │    └── heads/
 │         └── master       # Pointer to current commit hash
 └── HEAD                  # Reference to current branch
```

This structure ensures efficient storage and retrieval of Git objects by splitting SHA-1 hashes into directory and file components, minimizing filesystem overhead and lookup complexity.

---

This documentation provides a comprehensive reference for developers working on or extending the pygit object storage subsystem, facilitating a deep understanding of Git's object model and its practical implementation.