# refs.md

---

## Overview

The `refs.md` document provides detailed information on managing Git references and retrieving commit hashes for branches within the repository. This file fits into the broader "Commit and Branch Management" section of the repository documentation, where it complements related files like `commit.md` by focusing on how branch references are stored, accessed, and updated. Proper management of references is crucial for tracking branch histories, enabling operations such as committing, branching, and pushing changes. This document explains how references are handled internally, describes key functions that interact with branch references, and illustrates how to retrieve the commit hash currently pointed to by a branch like `master`.

---

## Function Documentation

### Function: `get_local_master_hash()`

**Purpose:**  
Retrieves the current commit hash (SHA-1 string) pointed to by the local `master` branch. This function reads the reference stored in `.git/refs/heads/master` to determine the latest commit on the branch.

**Parameters:**  
None

**Returns:**  
- A string representing the SHA-1 commit hash of the local `master` branch.  
- Returns `None` if the reference file is missing (e.g., no commits have been made yet).

**Operation Details:**  
1. Construct the path to the `master` branch reference file, which is `.git/refs/heads/master`.  
2. Attempt to read the contents of this file.  
3. Decode the contents from bytes to a string and strip any whitespace or newline characters.  
4. Return the resulting commit hash string.  
5. If the file does not exist (e.g., no commits made yet), catch the `FileNotFoundError` and return `None`.

**Example Usage:**

```python
commit_hash = get_local_master_hash()
if commit_hash:
    print(f"Current commit hash on local master branch: {commit_hash}")
else:
    print("No commits found on local master branch.")
```

---

### Function: `write_file(path, data)`

**Purpose:**  
Writes raw bytes (`data`) to a file specified by `path`. This utility is used internally when updating references or storing objects.

**Parameters:**  
- `path` (str): The file system path where data should be written.  
- `data` (bytes): The byte content to write to the file.

**Returns:**  
None

**Operation Details:**  
1. Opens the file at the specified `path` in binary write mode (`'wb'`).  
2. Writes the byte data to the file.  
3. Closes the file (handled automatically via `with` statement).

**Example Usage:**

```python
ref_path = ".git/refs/heads/master"
commit_hash = "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0"
write_file(ref_path, (commit_hash + "\n").encode())
print(f"Updated {ref_path} with commit hash {commit_hash}")
```

---

### Function: `hash_object(data, obj_type, write=True)`

**Purpose:**  
Calculates the SHA-1 hash of an object with given data and type (e.g., 'commit', 'tree', 'blob'), optionally writing the compressed object to the Git object store. This is essential for uniquely identifying Git objects and storing them properly.

**Parameters:**  
- `data` (bytes): The raw data of the object.  
- `obj_type` (str): The type of the Git object (e.g., `'commit'`, `'tree'`, `'blob'`).  
- `write` (bool, optional): If `True` (default), writes the compressed object to `.git/objects`. If `False`, only computes and returns the hash.

**Returns:**  
- A SHA-1 hash string that uniquely identifies the object.

**Operation Details:**  
1. Create a header string in the format `"{obj_type} {len(data)}"`, encode it as bytes.  
2. Concatenate the header, a NUL byte (`\x00`), and the data to form the full object representation.  
3. Compute the SHA-1 hash of this full data.  
4. If `write` is `True`, determine the storage path in `.git/objects` based on the hash prefix and suffix.  
5. If the object does not already exist, compress the full data with zlib and write it to the object store.  
6. Return the SHA-1 hex digest.

**Example Usage:**

```python
commit_data = b"tree abcd1234\nparent 1234abcd\nauthor Alice <alice@example.com> 1600000000 +0000\ncommitter Alice <alice@example.com> 1600000000 +0000\n\nInitial commit\n"
commit_hash = hash_object(commit_data, "commit")
print(f"Created commit object with SHA-1: {commit_hash}")
```

---

### Function: `commit(message, author=None)`

**Purpose:**  
Creates a new commit object from the current index state, writes it to the object store, and updates the `master` branch reference to point to the new commit. Returns the SHA-1 hash of the new commit.

**Parameters:**  
- `message` (str): The commit message describing the changes.  
- `author` (str, optional): The author identity in the format `"Name <email>"`. If `None`, the function uses environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Returns:**  
- SHA-1 hash string of the newly created commit.

**Operation Details:**  
1. Call `write_tree()` to write the current index state as a tree object and get its SHA-1 hash.  
2. Retrieve the current local master commit hash via `get_local_master_hash()` to set as the parent commit.  
3. Determine the author string and timestamp with timezone offset.  
4. Construct the commit object content including tree, parent (if any), author, committer, and message fields.  
5. Hash and write the commit object using `hash_object`.  
6. Update the `.git/refs/heads/master` file to point to the new commit hash using `write_file`.  
7. Print a confirmation message and return the commit SHA-1.

**Example Usage:**

```python
new_commit_hash = commit("Fix typo in README")
print(f"Committed changes. New commit hash: {new_commit_hash}")
```

---

### Function: `write_tree()`

**Purpose:**  
Creates a tree object from the current index entries, serializes it, hashes it, and writes it to the object store. This function prepares a snapshot of the working directory state for committing.

**Parameters:**  
None

**Returns:**  
- SHA-1 hash string of the written tree object.

**Operation Details:**  
1. Read all index entries using `read_index()`.  
2. For each entry, format its file mode and path as bytes, append a NUL byte, then append the SHA-1 hash of the object's blob.  
3. Concatenate all entry representations into a single bytes object.  
4. Hash and write the concatenated data as a tree object using `hash_object`.  
5. Return the tree SHA-1 hash.

**Example Usage:**

```python
tree_hash = write_tree()
print(f"Tree object created with SHA-1: {tree_hash}")
```

---

### ASCII Diagram: Reference Storage Structure

```
.git/
├── HEAD                  # Points to current branch (e.g., "ref: refs/heads/master")
├── refs/
│   ├── heads/
│   │   ├── master        # Contains SHA-1 of latest commit on master branch
│   │   └── feature-x     # SHA-1 of latest commit on feature-x branch
│   └── tags/
│       └── v1.0          # Tag references
└── objects/
    ├── aa/
    │   └── bbccddeeff... # Object files named by SHA-1 hashes
    └── ...
```

**Explanation:**  
- Branch references are stored as files under `.git/refs/heads/`.  
- Each branch file contains the SHA-1 hash string of the commit it points to.  
- The `HEAD` file points to the current branch reference, usually `refs/heads/master`.  
- When a commit is made, the corresponding reference file is updated with the hash of the new commit.

---

## Summary

This document has outlined the key components involved in managing Git references within the repository, focusing on retrieving and updating commit hashes for branches, particularly the `master` branch. The functions described enable reading the current commit SHA-1 from reference files, writing new references after commits, and ensuring that the internal Git structure reflects the latest repository state. Understanding these mechanisms is essential for implementing Git-like version control features and maintaining the integrity of branch histories.