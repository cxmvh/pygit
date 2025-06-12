# repository.md - Initializing Repository and Getting Commit Hashes

---

## Overview

This document details the processes involved in initializing a Git repository and obtaining commit hashes, primarily focusing on the `pygit.commit` code flow. It fits within the broader "Repository Initialization and Management" section of the project documentation, providing foundational knowledge for managing repository state and committing changes. The file covers key functions that interact with the Git object store, index, and references to enable committing to the `master` branch and retrieving commit hashes. Understanding these functions is critical for grasping how commits are created, stored, and tracked in a simplified Git implementation.

---

## Function Documentation

### 1. `commit(message, author=None)`

#### Purpose

Create a commit object representing the current state of the repository index and update the local `master` branch reference to point to this new commit. The function returns the SHA-1 hash of the newly created commit.

#### Parameters

- `message` (str): Commit message describing the changes.
- `author` (str, optional): Author information (format: `Name <email>`). If `None`, defaults to environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

#### Operation Steps

1. Write the current index state as a tree object using `write_tree()`.
2. Retrieve the current commit hash of the local `master` branch with `get_local_master_hash()`.
3. Determine the author string, defaulting to environment variables if none provided.
4. Create a timestamp and UTC offset string for commit metadata.
5. Construct the commit object lines, including tree, parent (if exists), author, committer, and message.
6. Encode and hash the commit data with `hash_object()`, storing it in the object store.
7. Update the `refs/heads/master` file to point to the new commit SHA-1.
8. Print a confirmation message and return the commit SHA-1.

#### Example Usage

```python
commit_hash = commit("Initial commit")
print(f"New commit created: {commit_hash}")
```

---

### 2. `get_local_master_hash()`

#### Purpose

Retrieve the current commit hash (SHA-1 string) pointed to by the local `master` branch reference.

#### Parameters

None

#### Operation Steps

1. Read the contents of `.git/refs/heads/master`.
2. Return the stripped SHA-1 string if the file exists.
3. If the reference file does not exist (e.g., no commits yet), return `None`.

#### Example Usage

```python
master_hash = get_local_master_hash()
if master_hash:
    print(f"Current master commit: {master_hash}")
else:
    print("No commits found on master.")
```

---

### 3. `write_tree()`

#### Purpose

Generate a tree object that represents the current index contents and write it to the object store. Returns the SHA-1 hash of the tree.

#### Parameters

None

#### Preconditions

- The function supports only a flat directory structure (no nested directories).
- The index must be populated with entries.

#### Operation Steps

1. Read all entries from the index via `read_index()`.
2. For each entry:
   - Format the mode and path as bytes.
   - Concatenate mode/path with the entry's SHA-1.
3. Combine all entries and hash as a `tree` object using `hash_object()`.
4. Return the SHA-1 of the tree object.

#### Example Usage

```python
tree_hash = write_tree()
print(f"Tree object created: {tree_hash}")
```

---

### 4. `hash_object(data, obj_type, write=True)`

#### Purpose

Compute the SHA-1 hash of the given object data and write the object to the Git object store if `write` is `True`.

#### Parameters

- `data` (bytes): Raw data of the Git object.
- `obj_type` (str): Type of Git object (e.g., `'commit'`, `'tree'`, `'blob'`).
- `write` (bool): Whether to persist the object data into the object store.

#### Operation Steps

1. Create a header combining the object type and data length.
2. Concatenate header, a null byte, and the object data.
3. Compute the SHA-1 hash of this full data.
4. If writing is enabled:
   - Determine the object path in `.git/objects/` based on the hash prefix.
   - Compress and write the object data if it does not already exist.
5. Return the SHA-1 hash as a hex string.

#### Example Usage

```python
data = b"Example blob data"
sha1 = hash_object(data, 'blob')
print(f"Object stored with SHA-1: {sha1}")
```

---

### 5. `write_file(path, data)`

#### Purpose

Write binary data to the specified file path, creating or overwriting the file.

#### Parameters

- `path` (str): File system path to write to.
- `data` (bytes): Binary data to write.

#### Operation Steps

1. Open the file at `path` in binary write mode.
2. Write the `data` bytes.
3. Close the file.

#### Example Usage

```python
write_file(".git/refs/heads/master", b"abc123def4567890\n")
```

---

### 6. `init(repo)`

#### Purpose

Initialize a new Git repository by creating the necessary directory structure and files in the specified `repo` directory.

#### Parameters

- `repo` (str): Path to the directory where the repository is initialized.

#### Operation Steps

1. Create the `repo` directory.
2. Inside `repo`, create `.git`, `.git/objects`, `.git/refs`, and `.git/refs/heads` directories.
3. Write the `HEAD` file pointing to `refs/heads/master`.
4. Print a confirmation message.

#### Example Usage

```python
init("my_new_repo")
```

---

### ASCII Diagram: Repository Directory Structure After Initialization

```
my_new_repo/
└── .git/
    ├── HEAD                 # points to refs/heads/master
    ├── objects/             # object store directory
    └── refs/
        └── heads/
            └── master       # file pointing to latest commit hash
```

---

### Summary Diagram: Commit Creation Flow

```
+----------------+        +----------------+        +------------------+
|    Index       |  -->   |  write_tree()  |  -->   |  Tree Object SHA1 |
+----------------+        +----------------+        +------------------+
                                                           |
                                                           v
+----------------+        +----------------+        +------------------+
| Parent Commit  |        | commit()       |  -->   | Commit Object SHA1|
| SHA-1 (optional)|       | creates commit |        | (stored in .git) |
+----------------+        +----------------+        +------------------+
                                                           |
                                                           v
+----------------+
| Update master  |
| ref with new   |
| commit SHA-1   |
+----------------+
```

---

# Summary

The `repository.md` file documents critical functions for initializing a Git repository and managing commits within the `pygit` project. It covers repository setup, object hashing, tree writing, commit creation, and updating references, providing a foundation for understanding how commits are created and referenced. The documentation includes usage examples and diagrams to clarify the flow of data and the repository structure.