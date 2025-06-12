# object_management.md

---

## Overview

This document provides detailed technical information on the handling of Git objects within the repository, focusing on the processes of hashing, reading, and writing fundamental Git objects including blobs, trees, and commits. It fits into the broader "Repository Initialization and Core Objects" section of the documentation tree and supports essential repository operations such as initialization (`init`), indexing, and committing.

Understanding these core object management functions is crucial since Git's content-addressable storage model relies on properly creating, reading, and storing these objects. This document explains the underlying implementations and usage patterns for working with these objects using the `pygit` library, enabling developers to grasp how Git internally manages data.

---

## Function Documentation

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of the given object data of a specific type (e.g., `blob`, `tree`, `commit`). Optionally writes the compressed object data to the Git object store in the `.git/objects` directory.

**Parameters:**  
- `data` (`bytes`): The raw content of the object.  
- `obj_type` (`str`): The type of the object, such as `'blob'`, `'tree'`, or `'commit'`.  
- `write` (`bool`, optional): Whether to write the object data to disk. Defaults to `True`.

**Operation Steps:**  
1. Construct the object header as `<obj_type> <len(data)>`, encoded as bytes.  
2. Concatenate the header, a null byte (`\x00`), and the actual data to form the full object data.  
3. Calculate the SHA-1 hash of the full object data.  
4. If `write` is `True`, compress the data using zlib and store it under `.git/objects/xx/yyyy...` where `xx` are the first two hex digits of the SHA-1, and `yyyy...` are the remaining digits.  
5. Return the SHA-1 hash as a hexadecimal string.

**Example Usage:**

```python
file_contents = b'Hello, Git!'
blob_sha1 = hash_object(file_contents, 'blob')
print(f"Blob SHA-1: {blob_sha1}")
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object from the `.git/objects` directory given a SHA-1 prefix and returns its type and raw data.

**Parameters:**  
- `sha1_prefix` (`str`): The prefix (or full) SHA-1 hex string identifying the object.

**Operation Steps:**  
1. Locate the object file by resolving the SHA-1 prefix to the full path in `.git/objects`.  
2. Read and decompress the object data using zlib.  
3. Parse the header to extract the object type and the data size.  
4. Return a tuple of `(object_type, data_bytes)`.

**Example Usage:**

```python
obj_type, data = read_object('e69de29bb2d1d6434b8b29ae775ad8c2e48c5391')
print(f"Object Type: {obj_type}")
print(f"Data: {data.decode()}")
```

---

### `write_file(path, data)`

**Purpose:**  
Writes raw byte data to a file at the specified path. Used internally for writing Git objects and references.

**Parameters:**  
- `path` (`str`): File system path where data is written.  
- `data` (`bytes`): Data to be written.

**Operation Steps:**  
1. Open the file in binary write mode.  
2. Write the bytes to the file.  
3. Close the file.

**Example Usage:**

```python
write_file('.git/refs/heads/master', b'abc123\n')
```

---

### `read_file(path)`

**Purpose:**  
Reads the contents of a file at a given path as bytes.

**Parameters:**  
- `path` (`str`): Path to the file.

**Operation Steps:**  
1. Open the file in binary read mode.  
2. Read and return the content.

**Example Usage:**

```python
data = read_file('.git/HEAD')
print(data.decode())
```

---

### `write_index(entries)`

**Purpose:**  
Serializes and writes a list of `IndexEntry` objects to the Git index file `.git/index`.

**Parameters:**  
- `entries` (`List[IndexEntry]`): List of index entries representing files staged for commit.

**Operation Steps:**  
1. Pack each index entry's metadata and path into a binary format using `struct`.  
2. Compute the length of each entry padded to an 8-byte boundary.  
3. Concatenate all entries and prepend the index header containing the signature (`DIRC`), version (2), and entry count.  
4. Append a SHA-1 checksum of the entire index data.  
5. Write the complete bytes to `.git/index`.

**Example Usage:**

```python
write_index(my_index_entries)
```

---

### `read_index()`

**Purpose:**  
Reads and parses the `.git/index` file, returning a list of `IndexEntry` objects.

**Returns:**  
- `List[IndexEntry]`: Parsed index entries.

**Operation Steps:**  
1. Read the full index file data.  
2. Validate the signature, version, and checksum.  
3. Iterate through entry data, unpacking each entry's fixed fields and path.  
4. Return the list of entries.

**Example Usage:**

```python
entries = read_index()
for e in entries:
    print(e.path, e.sha1.hex())
```

---

### `write_tree()`

**Purpose:**  
Creates a Git tree object from the current index entries representing the top-level directory and writes it to the object store.

**Returns:**  
- `str`: SHA-1 hash of the created tree object.

**Operation Steps:**  
1. Read the current index entries.  
2. For each entry, format the mode and path, followed by the SHA-1 of the blob.  
3. Concatenate all entries and hash as a `tree` type object via `hash_object`.

**Example Usage:**

```python
tree_sha1 = write_tree()
print(f"Tree SHA-1: {tree_sha1}")
```

---

### `commit(message, author=None)`

**Purpose:**  
Creates a commit object from the current index state and writes it, updating the `master` branch reference.

**Parameters:**  
- `message` (`str`): Commit message.  
- `author` (`str`, optional): Author string in the format `"Name <email>"`. Defaults to environment variables.

**Returns:**  
- `str`: SHA-1 hash of the commit object.

**Operation Steps:**  
1. Generate a tree object from the index.  
2. Get the parent commit SHA-1 from `refs/heads/master`.  
3. Compose commit metadata lines including tree, parent(s), author, committer, timestamps, and message.  
4. Hash the commit data as a `commit` object.  
5. Update `refs/heads/master` with the new commit SHA-1.  
6. Print confirmation and return the SHA-1.

**Example Usage:**

```python
commit_sha1 = commit("Initial commit")
print(f"New commit: {commit_sha1}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieves the current commit SHA-1 hash pointed to by the local `master` branch.

**Returns:**  
- `str` or `None`: SHA-1 hex string or `None` if no commit exists.

**Operation Steps:**  
1. Read the ref file at `.git/refs/heads/master`.  
2. Return the stripped SHA-1 or `None` if the file does not exist.

**Example Usage:**

```python
master_sha1 = get_local_master_hash()
print(f"Local master SHA-1: {master_sha1}")
```

---

### `get_status()`

**Purpose:**  
Determines the working copy status by comparing files in the working directory against the index.

**Returns:**  
- Tuple of lists: `(changed_paths, new_paths, deleted_paths)`.

**Operation Steps:**  
1. Walk the working directory, excluding `.git`, collecting all file paths.  
2. Read index entries and map them by path.  
3. Determine changed files by comparing blob hashes.  
4. Identify new files present in working directory but not in index.  
5. Identify deleted files present in index but missing from working directory.

**Example Usage:**

```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### `add(paths)`

**Purpose:**  
Adds one or more file paths to the Git index by hashing their contents and updating the index entries.

**Parameters:**  
- `paths` (`List[str]`): List of file paths to add.

**Operation Steps:**  
1. Normalize paths to use forward slashes.  
2. Read existing index entries and filter out any matching the new paths.  
3. For each new path:  
   - Read file contents and hash as a blob object.  
   - Stat the file to obtain metadata for index entry.  
   - Create and append a new `IndexEntry`.  
4. Sort entries by path and write the updated index.

**Example Usage:**

```python
add(['README.md', 'src/main.py'])
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a Git tree object to extract its entries (files and subtrees).

**Parameters:**  
- `sha1` (`str`, optional): SHA-1 of the tree object.  
- `data` (`bytes`, optional): Raw tree data bytes. Must provide either `sha1` or `data`.

**Returns:**  
- List of tuples: `(mode, path, sha1_hex)` representing each tree entry.

**Operation Steps:**  
1. If `sha1` is given, read and decompress the object data.  
2. Iterate through the data, splitting entries on null bytes.  
3. Parse mode and path, then extract the 20-byte SHA-1 digest.  
4. Return the list of parsed entries.

**Example Usage:**

```python
entries = read_tree(tree_sha1)
for mode, path, sha1 in entries:
    print(f"{oct(mode)} {path} {sha1}")
```

---

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively retrieves all object SHA-1 hashes referenced by a tree object, including blobs and nested trees.

**Parameters:**  
- `tree_sha1` (`str`): The SHA-1 of the tree to scan.

**Returns:**  
- `set` of SHA-1 strings for all referenced objects (trees and blobs).

**Operation Steps:**  
1. Add the tree SHA-1 to the result set.  
2. Read the tree entries.  
3. For each entry, if it's a directory (tree), recursively call `find_tree_objects`.  
4. Otherwise, add the blob SHA-1 to the set.

**Example Usage:**

```python
objects = find_tree_objects(tree_sha1)
print(f"All objects in tree: {objects}")
```

---

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively collects all object SHA-1 hashes that are reachable from a commit, including its tree and parent commits.

**Parameters:**  
- `commit_sha1` (`str`): The SHA-1 of the commit object.

**Returns:**  
- `set` of SHA-1 strings including the commit, its tree, parents, and all nested objects.

**Operation Steps:**  
1. Add the commit SHA-1 to the set.  
2. Read and parse the commit object.  
3. Extract the tree SHA-1 and add all objects from that tree recursively.  
4. For each parent commit, recursively add all their objects.  
5. Return the full set.

**Example Usage:**

```python
all_objects = find_commit_objects(commit_sha1)
print(f"Objects referenced by commit: {all_objects}")
```

---

### `ls_files(details=False)`

**Purpose:**  
Lists the files currently staged in the Git index, optionally showing detailed metadata.

**Parameters:**  
- `details` (`bool`, optional): If `True`, shows mode, SHA-1, and stage number.

**Operation Steps:**  
1. Read the index entries.  
2. If `details` is `True`, print each entry with mode, SHA-1, stage, and path.  
3. Otherwise, just print the file paths.

**Example Usage:**

```python
ls_files(details=True)
```

---

## ASCII Diagram: Git Object Storage Layout

```
.git/
└── objects/
    ├── xx/
    │   └── yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy (compressed object)
    ├── yy/
    │   └── zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz
    └── ...
```

- `xx` are the first two hex digits of the SHA-1 hash.
- `yyyy...` are the remaining 38 hex digits.
- Each file stores a zlib-compressed object consisting of a header (`<type> <size>\0`) and the raw data bytes.

---

## Summary

This document has detailed the internal mechanics and usage examples of the core Git object management functions implemented in `pygit`. Understanding these functions is vital for grasping how Git stores data reliably and efficiently through content-addressable storage, enabling version control operations such as committing, branching, and pushing.