# repository.md

# Initializing Repository and Getting Commit Hashes

---

## Overview

This document covers the functionality related to initializing a Git repository and retrieving commit hashes within the `pygit` project. This file fits within the broader "Repository Initialization and Management" section, providing detailed explanations on how the repository structure is set up and how commit objects are created and accessed. The ability to initialize a repository and manage commit hashes is fundamental for version control operations, enabling tracking of changes and history within the repository.

---

## Function Documentation

### `init(repo)`

#### Purpose

Initializes a new Git repository by creating the required directory structure inside the specified `repo` directory. This includes setting up the `.git` directory and essential subdirectories to hold objects and references.

#### Parameters

- `repo` (str): The path/name of the directory to initialize as a Git repository.

#### Detailed Operation

1. Creates the main repository directory.
2. Creates the `.git` directory inside the repository.
3. Creates the following subdirectories inside `.git`:
   - `objects` - Stores Git objects (blobs, trees, commits).
   - `refs` - Stores references.
   - `refs/heads` - Stores local branch heads.
4. Writes the `HEAD` file inside `.git` pointing to `refs/heads/master` to indicate the default branch.

#### Usage Example

```python
init('my_project')
# Output:
# initialized empty repository: my_project
```

#### ASCII Diagram

```
my_project/
└── .git/
    ├── objects/
    └── refs/
        └── heads/
    └── HEAD  (contains: ref: refs/heads/master)
```

---

### `get_local_master_hash()`

#### Purpose

Retrieves the current commit hash that the local `master` branch points to. Returns `None` if no commits exist yet.

#### Parameters

None

#### Detailed Operation

1. Constructs the path `.git/refs/heads/master`.
2. Attempts to read this file, which contains the SHA-1 hash of the latest commit.
3. If the file does not exist (i.e., no commits have been made), returns `None`.

#### Usage Example

```python
commit_hash = get_local_master_hash()
if commit_hash:
    print(f"Current commit hash on master: {commit_hash}")
else:
    print("No commits found on master branch.")
```

---

### `commit(message, author=None)`

#### Purpose

Creates a new commit object representing the current state of the index and updates the `master` branch reference to point to this new commit. Returns the SHA-1 hash of the created commit.

#### Parameters

- `message` (str): Commit message describing the changes.
- `author` (str, optional): Author information in the format `"Name <email>"`. If not provided, uses environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

#### Detailed Operation

1. Calls `write_tree()` to write the current index entries as a tree object, getting its SHA-1.
2. Retrieves the current commit hash of `master` using `get_local_master_hash()` to set as parent.
3. Constructs author/committer metadata with timestamp and timezone offset.
4. Assembles commit object content:
   - `tree <tree_sha1>`
   - `parent <parent_sha1>` (if parent exists)
   - `author <author> <timestamp> <timezone>`
   - `committer <author> <timestamp> <timezone>`
   - Blank line
   - Commit message
   - Blank line
5. Hashes and writes the commit object using `hash_object`.
6. Updates `.git/refs/heads/master` with the new commit's SHA-1.
7. Prints confirmation and returns the commit SHA-1.

#### Usage Example

```python
commit_hash = commit("Initial commit")
print(f"Committed with hash: {commit_hash}")
```

---

### Supporting Functions

#### `write_tree()`

Creates a tree object from the current index entries (files staged for commit).

- Iterates over index entries.
- For each entry, formats mode, path, and SHA-1.
- Joins all entries and hashes as a tree object.

Returns the SHA-1 of the tree object.

#### Example

```python
tree_hash = write_tree()
print(f"Tree object hash: {tree_hash}")
```

#### `write_file(path, data)`

Writes binary `data` to the specified file path.

#### Example

```python
write_file('.git/refs/heads/master', b'commit_hash\n')
```

#### `hash_object(data, obj_type, write=True)`

Computes SHA-1 hash of the object data of type `obj_type` (e.g., "commit", "tree", "blob").

- Optionally writes the compressed object to the `.git/objects` directory.
- Returns the hexadecimal SHA-1 hash string.

#### Example

```python
sha1 = hash_object(b"example data", "blob")
print(f"Object SHA-1: {sha1}")
```

---

## ASCII Diagram: Commit Object Structure

```
+----------------------------+
| "tree <tree_sha1>"         |
| "parent <parent_sha1>"     |  <- optional, if previous commit exists
| "author <author> <time>"   |
| "committer <author> <time>"|
|                            |
| <commit message>           |
+----------------------------+
```

The commit object is stored compressed in the Git object store under `.git/objects`.

---

## Summary

This documentation covers the core repository initialization and commit creation workflows in `pygit`. It explains how to set up the repository structure, write tree and commit objects, and update branch references accordingly. Understanding these functions is critical for implementing or extending Git-like version control behavior.