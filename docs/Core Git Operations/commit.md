# commit.md

## Overview

The `commit.md` file documents the details of creating commits and the related functions within the `pygit` project. It resides under the "Core Git Operations" section of the project's documentation tree, indicating its central role in managing one of Git's fundamental features: committing changes to the repository. This document covers how commits are created from the current index state, how commit objects are formed and stored, and various helper functions that support commit creation and management. Understanding this file is essential for grasping how pygit handles the transition from staged changes to recorded repository history.

---

## Function Documentation

### `commit(message, author=None)`

#### Purpose
Creates a new commit object in the repository representing the current state of the index. The commit is recorded on the `master` branch with the provided commit message and optional author information. If no author is specified, it defaults to the environment's configured author name and email.

#### Parameters
- `message` (str): The commit message describing the change.
- `author` (str, optional): The author string in the format `"Name <email>"`. If omitted, derived from environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

#### Operation Steps
1. Calls `write_tree()` to create a tree object from the current index, representing the snapshot of staged files.
2. Retrieves the current commit hash of the local `master` branch using `get_local_master_hash()` to set the parent commit.
3. If no author is specified, reads author information from environment variables.
4. Computes the author timestamp and UTC offset to embed in the commit metadata.
5. Constructs the commit object content including:
   - `tree` line with the tree hash.
   - `parent` line if a parent commit exists.
   - `author` and `committer` lines with author and timestamp.
   - Blank line followed by the commit message.
6. Hashes and writes the commit object to the object store using `hash_object()`.
7. Updates the `refs/heads/master` reference to point to the new commit.
8. Prints confirmation and returns the commit SHA-1 hash.

#### Example Usage
```python
commit_hash = commit("Add README and initial project files")
print(f"New commit created: {commit_hash}")
```

---

### `write_tree()`

#### Purpose
Writes a Git tree object representing the current index's staged files. This tree encodes file modes, paths, and their corresponding blob hashes.

#### Parameters
None

#### Operation Steps
1. Reads the current index entries via `read_index()`.
2. For each index entry:
   - Asserts that file paths do not contain subdirectories (current limitation).
   - Formats the mode and path as bytes.
   - Concatenates mode/path and SHA-1 of the blob as a tree entry.
3. Joins all tree entries and writes a tree object using `hash_object()`.
4. Returns the SHA-1 hash of the written tree object.

#### Example Usage
```python
tree_hash = write_tree()
print(f"Tree object created with hash: {tree_hash}")
```

---

### `get_local_master_hash()`

#### Purpose
Retrieves the SHA-1 hash of the current commit pointed to by the local `master` branch.

#### Parameters
None

#### Operation Steps
1. Reads the contents of `.git/refs/heads/master`.
2. If the reference file doesn't exist (e.g., no commits yet), returns `None`.
3. Otherwise, returns the commit hash string.

#### Example Usage
```python
current_master = get_local_master_hash()
if current_master:
    print(f"Current master commit: {current_master}")
else:
    print("No commits on master branch yet.")
```

---

### `hash_object(data, obj_type, write=True)`

#### Purpose
Generates the SHA-1 hash of a Git object of a specified type (blob, tree, commit) with given data. Optionally writes the compressed object to the `.git/objects` directory.

#### Parameters
- `data` (bytes): Raw content of the object.
- `obj_type` (str): The type of object (e.g., `"blob"`, `"tree"`, `"commit"`).
- `write` (bool): If `True`, writes the object to disk.

#### Operation Steps
1. Creates a header string with format `"{obj_type} {len(data)}"`.
2. Concatenates header, a null byte, and data.
3. Calculates SHA-1 hash of the full object.
4. If `write` is `True`, compresses and writes the object to `.git/objects/xx/yyyyyy`.
5. Returns the SHA-1 hex string.

#### Example Usage
```python
blob_hash = hash_object(b"Hello, World!\n", "blob")
print(f"Blob object hash: {blob_hash}")
```

---

### `write_file(path, data)`

#### Purpose
Writes raw byte data to a file at the specified path.

#### Parameters
- `path` (str): File path to write.
- `data` (bytes): Data to write.

#### Operation Steps
1. Opens the file in binary write mode.
2. Writes the data bytes.
3. Closes the file.

#### Example Usage
```python
write_file('.git/refs/heads/master', b'abc123\n')
```

---

### ASCII Diagram: Commit Object Structure

```
+----------------------------+
| tree <tree_hash>            |  <-- Reference to tree object
+----------------------------+
| parent <parent_hash>        |  <-- Optional: previous commit
+----------------------------+
| author <author> <timestamp> |
+----------------------------+
| committer <author> <timestamp> |
+----------------------------+
|                            |
| <commit message>           |
|                            |
+----------------------------+
```

---

### Additional Related Functions

Below is a summary of other functions involved in the commit process and their roles:

- **`read_index()`**: Reads the Git index file and returns a list of `IndexEntry` objects representing staged files.
- **`write_index(entries)`**: Writes a list of `IndexEntry` objects back to the `.git/index` file, maintaining the index state.
- **`get_status()`**: Compares the working directory and index to identify changed, new, and deleted files.
- **`add(paths)`**: Adds specified files to the index by hashing their contents and updating the index entries.
- **`read_object(sha1_prefix)`**: Reads a Git object by its SHA-1 prefix, returning its type and data.
- **`ls_files(details=False)`**: Lists files currently staged in the index, optionally showing detailed metadata.

---

### Example Commit Workflow Using pygit Functions

```python
from pygit import add, commit, status

# Stage files for commit
add(['README.md', 'setup.py'])

# Show current status
status()

# Commit staged changes with message and author
commit_hash = commit("Initial commit with README and setup", author="Alice <alice@example.com>")

print(f"Commit created: {commit_hash}")
```

---

## Summary

This document provides a comprehensive reference for the commit creation process within the pygit project. It explains how the current index state is captured as a tree object, how commit metadata is constructed, and how commit objects are stored and referenced in the repository. Understanding these components is essential for working with Git operations at a low level and extending or debugging pygit's commit functionality.