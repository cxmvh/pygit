# commit.md

# Commit Creation Process and Related Functions

---

## Overview

This document details the commit creation process within the pygit repository implementation and describes the key functions involved in creating commits, managing commit objects, and interacting with the Git index and object store. It fits within the broader "Commit and Object Management" section of the documentation tree, which focuses on handling commits, hashing objects, and managing Git trees and the index.

The commit process is central to version control, as commits represent snapshots of the repository state. This file explains how the commit command assembles the current state of the working directory into a commit object, writes trees, manages parent commits, and updates references, specifically for the `master` branch. Additionally, it covers supporting functions such as hashing objects, writing files, and reading/writing the index.

---

## Function Documentation

### 1. `commit(message, author=None)`

**Purpose:**  
Creates a new commit object representing the current state of the index and updates the `master` branch to point to this new commit. It returns the SHA-1 hash of the created commit.

**Parameters:**  
- `message` (str): Commit message describing the changes.  
- `author` (str, optional): Author identity string in the format `"Name <email>"`. If not provided, it is derived from environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Operation Steps:**  
1. Calls `write_tree()` to write the current index entries as a tree object and get its SHA-1 hash.  
2. Retrieves the current `master` branch commit hash by calling `get_local_master_hash()`. This will be the parent commit.  
3. If no author is provided, constructs the author string from environment variables.  
4. Computes the current timestamp and UTC offset to form the author timestamp string.  
5. Constructs the commit object contents including:  
   - `tree` line with tree hash  
   - `parent` line (if a parent exists)  
   - `author` and `committer` lines with author identity and timestamp  
   - Blank line  
   - Commit message  
6. Encodes the commit data and calls `hash_object(data, 'commit')` to write the commit object to the object store, getting back its SHA-1.  
7. Updates the `master` branch reference file with the new commit SHA-1.  
8. Prints confirmation and returns the commit SHA-1 string.

**Example Usage:**

```python
sha1 = commit("Add initial project files")
print(f"Created commit {sha1}")
```

---

### 2. `write_tree()`

**Purpose:**  
Creates a tree object from the current index entries, representing the state of files staged for commit.

**Parameters:**  
None

**Operation Steps:**  
1. Reads the current index entries using `read_index()`.  
2. For each index entry, asserts that it is a top-level file (no slashes in path supported currently).  
3. Formats each entry as `<mode> <path>\0<sha1>`, where mode is in octal, path is filename, and sha1 is the binary SHA-1 of the blob object.  
4. Concatenates all entries and calls `hash_object()` with object type `'tree'` to write the tree object.  
5. Returns the SHA-1 hash of the new tree object.

**Example Usage:**

```python
tree_sha1 = write_tree()
print(f"Tree object created with SHA-1: {tree_sha1}")
```

---

### 3. `get_local_master_hash()`

**Purpose:**  
Retrieves the current commit hash that the local `master` branch points to.

**Parameters:**  
None

**Operation Steps:**  
1. Reads the `.git/refs/heads/master` file content.  
2. Returns the SHA-1 string if the file exists; otherwise returns `None` indicating no commits yet.

**Example Usage:**

```python
current_master = get_local_master_hash()
if current_master:
    print(f"Local master points to commit {current_master}")
else:
    print("No commits on local master branch yet")
```

---

### 4. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of a Git object of given type and optionally writes the compressed object data to the `.git/objects` directory.

**Parameters:**  
- `data` (bytes): Raw content of the object.  
- `obj_type` (str): Type of object (`'blob'`, `'tree'`, `'commit'`, etc.).  
- `write` (bool): Whether to write the object to storage (default `True`).

**Operation Steps:**  
1. Prepares the object header as `<obj_type> <length>\0`.  
2. Concatenates header and data.  
3. Computes SHA-1 hash of the full data.  
4. If `write` is `True`, compresses and writes the data under `.git/objects/` using the first two characters of hash as directory and remaining as filename.  
5. Returns the SHA-1 hex string.

**Example Usage:**

```python
blob_sha1 = hash_object(b"Hello, Git!", "blob")
print(f"Blob object stored with hash: {blob_sha1}")
```

---

### 5. `write_file(path, data)`

**Purpose:**  
Writes raw byte data to the specified file path.

**Parameters:**  
- `path` (str): File path to write to.  
- `data` (bytes): Data bytes to write.

**Operation Steps:**  
1. Opens file at `path` in binary write mode.  
2. Writes the data bytes to the file.  
3. Closes the file.

**Example Usage:**

```python
write_file(".git/refs/heads/master", (sha1 + "\n").encode())
```

---

### 6. `read_index()`

**Purpose:**  
Reads the Git index file and parses it into a list of `IndexEntry` objects representing staged files.

**Parameters:**  
None

**Operation Steps:**  
1. Reads the `.git/index` file contents.  
2. Verifies the SHA-1 checksum of the index data.  
3. Parses the header (`DIRC` signature, version, number of entries).  
4. Iterates over entries, unpacking metadata and path, creating `IndexEntry` instances.  
5. Returns the list of index entries.

**Example Usage:**

```python
index_entries = read_index()
for entry in index_entries:
    print(f"Staged file: {entry.path}, mode: {oct(entry.mode)}")
```

---

### 7. `add(paths)`

**Purpose:**  
Adds one or more files to the Git index, hashing their contents and updating the index entries.

**Parameters:**  
- `paths` (list of str): List of file paths to add.

**Operation Steps:**  
1. Normalizes paths to use forward slashes.  
2. Reads the current index entries, excluding any with paths being added (to update them).  
3. For each path:  
   - Reads file contents and hashes as a blob object.  
   - Collects file metadata (`stat` info).  
   - Creates a new `IndexEntry` with this data and the blob SHA-1.  
4. Combines old and new entries, sorts them by path.  
5. Writes updated entries to index using `write_index()`.

**Example Usage:**

```python
add(["README.md", "src/main.py"])
print("Files added to index")
```

---

### ASCII Diagram: Commit Object Structure

```
+-------------------------------------------------------+
| commit object (text)                                  |
+-------------------------------------------------------+
| tree <tree_sha1>                                      |
| parent <parent_sha1> (optional)                       |
| author <author_name> <author_email> <timestamp>      |
| committer <committer_name> <committer_email> <timestamp> |
|                                                       |
| <commit message>                                      |
+-------------------------------------------------------+
```

This textual commit object is hashed and stored as a Git commit object. The `commit()` function automates this creation and updates branch references.

---

### 8. `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the Git index file `.git/index`.

**Parameters:**  
- `entries` (list of `IndexEntry`): Index entries to write.

**Operation Steps:**  
1. Packs each entry's metadata fields into binary format.  
2. Calculates aligned length for each entry including path and padding.  
3. Combines all packed entries with the index file header.  
4. Computes SHA-1 checksum of the entire index data (excluding the checksum itself).  
5. Appends checksum and writes to `.git/index`.

**Example Usage:**

```python
write_index(updated_entries)
print("Index file written with updated entries")
```

---

### Summary

The commit creation process in pygit involves:

1. Staging files using `add()`.  
2. Writing a tree object representing the index using `write_tree()`.  
3. Creating a commit object with metadata and parent references using `commit()`.  
4. Hashing and writing objects with `hash_object()`.  
5. Updating branch references with `write_file()`.

These functions collectively enable the core commit functionality of the Git system implemented in pygit.

---

*End of commit.md documentation*