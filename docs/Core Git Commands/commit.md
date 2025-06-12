# commit.md

# Commit Command and Related Internals

---

## Overview

The `commit.md` document provides comprehensive technical documentation for the `commit` command and its related internal functions within the `pygit` project. It is part of the broader "Core Git Commands" section, specifically nested under "Commit and Branch Management." This file explains how changes staged in the Git index are committed to the local repository, how commit objects are constructed and stored, and how the master branch pointer is updated accordingly.

This documentation is essential for understanding the implementation details of committing changes in `pygit`, how tree objects are created from the index, and how commit objects reference parent commits. It also covers the interaction between commit creation and object storage mechanisms.

---

## Function Descriptions

### 1. `commit(message, author=None)`

**Purpose:**  
Create a new commit object representing the current state of the index, write it to the object store, update the master branch reference, and return the SHA-1 hash of the commit.

**Parameters:**  
- `message` (str): The commit message describing the changes.  
- `author` (str, optional): The author string in the format `"Name <email>"`. If not provided, it is constructed from environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Operation:**  
1. Calls `write_tree()` to write the current index as a tree object and get its SHA-1 hash.  
2. Retrieves the current commit hash of the local master branch via `get_local_master_hash()`. This becomes the parent commit if present.  
3. Constructs the author and committer timestamp with timezone offset.  
4. Assembles the commit object content lines:
   - `tree <tree_sha1>`  
   - `parent <parent_sha1>` (if parent exists)  
   - `author <author> <timestamp>`  
   - `committer <author> <timestamp>`  
   - Blank line  
   - Commit message  
   - Blank line  
5. Encodes and hashes this data as a commit object using `hash_object()`.  
6. Writes the new commit SHA-1 to `.git/refs/heads/master` to update the master branch pointer.  
7. Prints confirmation and returns the commit SHA-1.

**Example Usage:**  
```python
commit_hash = commit("Initial commit")
print(f"Commit created with hash: {commit_hash}")
```

---

### 2. `write_tree()`

**Purpose:**  
Construct a tree object from the current index entries and write it to the object store. Returns the SHA-1 hash of the tree object.

**Parameters:**  
None

**Operation:**  
1. Reads the current index entries using `read_index()`.  
2. For each index entry, verifies it is a top-level file (no subdirectories supported yet).  
3. For each file, formats mode and path as bytes, appends the file's SHA-1 hash.  
4. Concatenates all entries and hashes them as a tree object using `hash_object()`.  
5. Returns the SHA-1 hash of the created tree.

**Example Usage:**  
```python
tree_hash = write_tree()
print(f"Tree object created with hash: {tree_hash}")
```

---

### 3. `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit SHA-1 hash pointed to by the local master branch.

**Parameters:**  
None

**Operation:**  
1. Reads the `.git/refs/heads/master` file.  
2. Returns the trimmed SHA-1 string if present, otherwise returns `None` if the master reference does not exist.

**Example Usage:**  
```python
current_master = get_local_master_hash()
if current_master:
    print(f"Current master commit: {current_master}")
else:
    print("No commits yet on master.")
```

---

### 4. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash of a Git object with given type and data. Optionally write the compressed object data to the Git object store.

**Parameters:**  
- `data` (bytes): Raw data of the object.  
- `obj_type` (str): Type of Git object (e.g., `'commit'`, `'tree'`, `'blob'`).  
- `write` (bool): Whether to write the object to disk. Defaults to `True`.

**Operation:**  
1. Constructs the Git object header: `<obj_type> <data_length>\0`.  
2. Concatenates header and data, calculates SHA-1 hash.  
3. If `write` is `True`, stores the compressed data in `.git/objects/` using the first two characters as directory and the remainder as filename.  
4. Returns the SHA-1 hash as a hex string.

**Example Usage:**  
```python
blob_sha1 = hash_object(b'Hello, world!\n', 'blob')
print(f"Blob object SHA-1: {blob_sha1}")
```

---

### 5. `write_file(path, data)`

**Purpose:**  
Write bytes data to a specified file path.

**Parameters:**  
- `path` (str): File path to write to.  
- `data` (bytes): Data bytes to write.

**Operation:**  
1. Opens the file in binary write mode.  
2. Writes the data bytes.

**Example Usage:**  
```python
write_file('.git/refs/heads/master', b'abc123\n')
```

---

### 6. `read_index()`

**Purpose:**  
Read the Git index file and return a list of `IndexEntry` objects representing the index contents.

**Parameters:**  
None

**Operation:**  
1. Reads the `.git/index` file contents as bytes.  
2. Verifies the SHA-1 checksum of the index data.  
3. Parses the header to confirm signature and version.  
4. Iteratively parses each index entry, decoding fields and file path.  
5. Returns the list of entries.

**Example Usage:**  
```python
index_entries = read_index()
for entry in index_entries:
    print(entry.path, entry.sha1.hex())
```

---

### 7. `add(paths)`

**Purpose:**  
Add specified file paths to the Git index, hashing their contents and updating the index.

**Parameters:**  
- `paths` (list of str): List of file paths to add.

**Operation:**  
1. Normalizes paths to use forward slashes.  
2. Reads existing index entries, excludes any entries matching paths to add.  
3. For each path to add:  
   - Reads file contents and hashes as blob object with `hash_object()`.  
   - Retrieves file stats for metadata.  
   - Creates a new `IndexEntry` with metadata and SHA-1.  
4. Sorts entries by path and writes updated index with `write_index()`.

**Example Usage:**  
```python
add(['README.md', 'src/main.py'])
```

---

### 8. `write_index(entries)`

**Purpose:**  
Write a list of `IndexEntry` objects to the `.git/index` file.

**Parameters:**  
- `entries` (list of IndexEntry): List of index entries to write.

**Operation:**  
1. Packs each `IndexEntry` into binary format according to Git index specifications.  
2. Constructs the index header with signature, version, and entry count.  
3. Concatenates all packed entries and header.  
4. Computes SHA-1 checksum of all preceding data and appends it.  
5. Writes the complete data to `.git/index`.

**Example Usage:**  
```python
write_index(index_entries)
```

---

### 9. `read_file(path)`

**Purpose:**  
Read the entire contents of a file as bytes.

**Parameters:**  
- `path` (str): Path of the file to read.

**Operation:**  
1. Opens the file in binary read mode.  
2. Reads and returns the data bytes.

**Example Usage:**  
```python
data = read_file('README.md')
print(data.decode())
```

---

## ASCII Diagram: Commit Object Structure and Flow

```
+----------------------------------------------+
|                  Commit Object                |
+----------------------------------------------+
| tree <tree_sha1>                              |
| parent <parent_sha1> (optional)               |
| author <author_name> <timestamp>              |
| committer <author_name> <timestamp>           |
|                                              |
| <commit message>                              |
+----------------------------------------------+

     |
     | (hash_object writes commit to .git/objects/)
     v

.git/objects/xx/yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
     |
     | (SHA-1 hash stored)
     v

.git/refs/heads/master  <-- updated to new commit SHA-1
```

---

## Summary

This documentation file describes the key internals of the `commit` command in `pygit`, including how the commit and tree objects are created from the index, how the master branch reference is updated, and how files are added and indexed. Understanding these functions is critical for extending or debugging commit behavior in this Git implementation.

For related functionality, see also:  
- `write_tree()` for tree object creation  
- `hash_object()` for object hashing and storage  
- `read_index()` and `write_index()` for index file management  
- `add()` for staging files before commit  

---

# End of commit.md documentation file content.