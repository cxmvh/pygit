# trees_and_commits.md

# Tree and Commit Management

## Overview

This document provides comprehensive technical documentation for the core operations related to Git tree and commit objects within the pygit project. It covers essential functions such as `read_tree`, `write_tree`, `find_tree_objects`, `find_commit_objects`, and `commit`, which are responsible for reading, writing, and traversing Git trees and commits.

Positioned under the **Tree and Commit Management** section of the broader documentation tree, this file plays a crucial role in explaining how the repository's directory structure and history are represented, manipulated, and persisted. These operations form the foundation for tracking file snapshots and project history, enabling functionalities like listing files, creating commits, and managing repository states.

The functions documented here are integral to the `pygit.ls_files` code flow and interact closely with other components such as object handling, index management, and repository utilities.

---

## Function Documentation

---

### `read_tree`

**Purpose:**  
Reads a Git tree object, parsing its contents into a list of entries describing the files and subdirectories contained within the tree.

**Signature:**  
```python
def read_tree(sha1=None, data=None) -> list[tuple[int, str, str]]:
```

**Parameters:**  
- `sha1` (str, optional): The SHA-1 hash of the tree object to read. Mutually exclusive with `data`.  
- `data` (bytes, optional): Raw bytes of the tree object data. Mutually exclusive with `sha1`.

**Returns:**  
- List of tuples `(mode, path, sha1)` where:  
  - `mode` is the file mode (e.g., 100644 for a file, 40000 for a directory).  
  - `path` is the filename or directory name.  
  - `sha1` is the SHA-1 hash of the object at that path.

**Operation:**  
- If `sha1` is provided, the function reads the tree object from the object store using `read_object`.  
- If `data` is provided directly, it uses that data for parsing.  
- The tree object data format consists of multiple entries. Each entry is:  
  ```
  <mode> <path>\0<20-byte SHA-1>
  ```  
- The function iteratively parses entries until no more null bytes are found, collecting mode, path, and SHA-1 hash for each entry.

**Example Usage:**
```python
# Read tree by SHA-1 hash
tree_entries = read_tree(sha1='a1b2c3d4e5f6...')

for mode, path, sha1 in tree_entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

---

### `write_tree`

**Purpose:**  
Creates a Git tree object from the current index entries, writing a snapshot of the working directory state.

**Signature:**  
```python
def write_tree() -> str:
```

**Returns:**  
- The SHA-1 hash (hex string) of the newly created tree object.

**Operation:**  
- Reads entries from the index file representing the current working directory state.  
- For each entry, constructs a tree entry of the form:  
  ```
  <mode> <path>\0<20-byte SHA-1>
  ```  
- Currently, only supports entries in the top-level directory (no nested subdirectories).  
- Concatenates all entries and hashes them as a Git `tree` object using `hash_object`.

**Example Usage:**
```python
tree_sha1 = write_tree()
print(f"Created tree object with SHA-1: {tree_sha1}")
```

---

### `find_tree_objects`

**Purpose:**  
Recursively collects all object SHA-1 hashes contained within a tree, including blobs and nested trees, plus the tree itself.

**Signature:**  
```python
def find_tree_objects(tree_sha1: str) -> set[str]:
```

**Parameters:**  
- `tree_sha1` (str): SHA-1 hash of the tree to analyze.

**Returns:**  
- Set of SHA-1 hashes (strings) representing all objects contained within the tree and its subtrees.

**Operation:**  
- Starts with the given tree SHA-1 in the result set.  
- Reads the tree entries using `read_tree`.  
- For each entry, if it is a directory (`mode` indicates directory), recursively calls `find_tree_objects` on that subtree and merges results.  
- Otherwise, adds the blob's SHA-1 to the set.

**Example Usage:**
```python
all_objects = find_tree_objects(tree_sha1='a1b2c3d4...')
print(f"Total objects found in tree: {len(all_objects)}")
```

**ASCII Diagram:**

```
Tree SHA-1 (root)
├── Blob SHA-1 (file1)
├── Blob SHA-1 (file2)
└── Tree SHA-1 (subdir)
    ├── Blob SHA-1 (file3)
    └── Blob SHA-1 (file4)
```

---

### `find_commit_objects`

**Purpose:**  
Recursively finds all objects (commits, trees, blobs) reachable from a commit, including its parents.

**Signature:**  
```python
def find_commit_objects(commit_sha1: str) -> set[str]:
```

**Parameters:**  
- `commit_sha1` (str): SHA-1 hash of the commit to analyze.

**Returns:**  
- Set of SHA-1 hashes of all objects reachable from the commit, including the commit itself, its tree, all blobs, and all parent commits recursively.

**Operation:**  
- Starts with the commit SHA-1 in the result set.  
- Reads the commit object using `read_object`.  
- Parses commit data to find the `tree` line and all `parent` lines.  
- Recursively calls `find_tree_objects` on the tree SHA-1 and merges results.  
- For each parent commit SHA-1 found, recursively calls `find_commit_objects` and merges results.

**Example Usage:**
```python
commit_objects = find_commit_objects(commit_sha1='deadbeef...')
print(f"Objects reachable from commit: {len(commit_objects)}")
```

**ASCII Diagram:**

```
Commit SHA-1
├── Tree SHA-1
│   ├── Blob SHA-1 (file1)
│   └── Tree SHA-1 (subdir)
│       └── Blob SHA-1 (file2)
└── Parent Commit SHA-1
    └── ...
```

---

### `commit`

**Purpose:**  
Creates a new commit object representing the current state of the index and updates the `master` branch reference.

**Signature:**  
```python
def commit(message: str, author: Optional[str] = None) -> str:
```

**Parameters:**  
- `message` (str): Commit message describing the changes.  
- `author` (str, optional): Author string in the format `"Name <email>"`. If not provided, uses environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Returns:**  
- SHA-1 hash string of the newly created commit object.

**Operation:**  
- Writes a tree object from the current index via `write_tree()`.  
- Retrieves the current commit hash of `master` branch as the parent.  
- Constructs the author and committer lines with timestamp and timezone offset.  
- Builds commit content lines including tree, parent (if any), author, committer, and message.  
- Hashes this content as a `commit` object and writes it to the object store.  
- Updates `.git/refs/heads/master` with the new commit SHA-1.  
- Prints confirmation message and returns the commit SHA-1.

**Example Usage:**
```python
new_commit_sha1 = commit("Add initial project files")
print(f"New commit created: {new_commit_sha1}")
```

**Detailed Workflow:**

```
+------------------+
| Write tree object | <-- current index -> tree SHA-1
+------------------+
         |
         v
+--------------------------+
| Get current master commit | <-- parent SHA-1
+--------------------------+
         |
         v
+----------------------------------+
| Build commit content lines:       |
| - tree <tree_sha1>                |
| - parent <parent_sha1> (optional)|
| - author <author> <timestamp>    |
| - committer <author> <timestamp> |
|                                  |
| - <commit message>               |
+----------------------------------+
         |
         v
+--------------------------+
| Hash and write commit obj |
+--------------------------+
         |
         v
+---------------------------------------+
| Update master ref to new commit SHA-1 |
+---------------------------------------+
```

---

## Summary

This document covers the foundational operations needed to manage Git tree and commit objects, including reading tree contents, writing tree snapshots from the index, recursively finding objects within trees and commits, and creating new commits that update the repository's history.

Together, these functions enable the core version control capabilities of pygit, supporting file tracking, history traversal, and repository state management.