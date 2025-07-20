---
sidebar_position: 3
---

# Commit Process and Supporting Functions

## Overview

This documentation covers the commit process within the pygit toolset, detailing how commits are created, trees are written, objects are hashed, references updated, and author information handled. It serves as a critical part of the **Change Workflows** section, providing developers with an in-depth understanding of how changes in the working directory and index are encapsulated into commits. The file also explains related utility functions such as `write_tree`, `hash_object`, `write_file`, `get_local_master_hash`, and `write_index` that support the commit operation.

---

## `commit`

### Purpose

The `commit` function is responsible for creating a new Git commit object in the repository. It takes the current state of the index (staged changes), writes the tree object representing the snapshot of the working directory, creates the commit object containing metadata and references to the tree and parent commit(s), and updates the HEAD reference to point to the new commit.

### Parameters

- `message` (str): The commit message describing the changes.
- `author` (str): The author information in the format `"Name <email>"`.
- `repo_dir` (str): Path to the root directory of the Git repository.

### Operation Steps

1. **Write Tree:** Calls `write_tree` to generate a tree object corresponding to the current index.
2. **Get Parent Commit:** Uses `get_local_master_hash` to retrieve the current commit hash pointed to by `refs/heads/master` (or the current branch).
3. **Create Commit Content:** Composes the commit content including:
    - Tree hash
    - Parent commit hash (if any)
    - Author information with timestamp
    - Commit message
4. **Hash Commit Object:** Uses `hash_object` to hash and write the commit object to the Git object database.
5. **Update Reference:** Updates the reference (e.g., `refs/heads/master`) to point to the new commit hash.
6. **Write Index:** Optionally updates the index file to reflect the committed state.

### Example Usage

```python
commit_message = "Fix bug in file parsing logic"
author_info = "Alice Example <alice@example.com>"
repository_path = "/path/to/myrepo"

commit(message=commit_message, author=author_info, repo_dir=repository_path)
```

---

## `write_tree`

### Purpose

Generates a Git tree object that represents the current state of the index (staged files and directories). This tree object is a snapshot of the project directory at commit time.

### Parameters

- `repo_dir` (str): Path to the root of the Git repository.

### Description

- Reads the current index file.
- For each entry, composes a tree entry line including the file mode, filename, and object hash.
- Recursively writes subtrees for directories.
- Writes the tree object to the object store using `hash_object` and returns the tree hash.

### Example Usage

```python
tree_hash = write_tree(repo_dir="/path/to/myrepo")
print(f"Tree object created: {tree_hash}")
```

---

## `hash_object`

### Purpose

Creates a Git object (blob, tree, or commit) by hashing the content with the appropriate header and writing it to the `.git/objects` directory.

### Parameters

- `data` (bytes): The raw content to be hashed and stored.
- `obj_type` (str): The type of the object, one of `"blob"`, `"tree"`, or `"commit"`.

### Operation Steps

1. Prepend a header in the format: `"{obj_type} {len(data)}\0"`.
2. Compute the SHA-1 hash of the header concatenated with the data.
3. Compress and write the object file into `.git/objects/` under directories named by the first two hex digits of the hash.
4. Return the hex SHA-1 hash string.

### Example Usage

```python
content = b"Hello, Git!"
blob_hash = hash_object(data=content, obj_type="blob")
print(f"Blob object hash: {blob_hash}")
```

---

## `write_file`

### Purpose

Writes data to a file at a specified path, ensuring atomicity and correctness.

### Parameters

- `path` (str): The file path where data should be written.
- `data` (bytes): The content to write to the file.

### Description

- Opens the file in binary write mode.
- Writes the data.
- Flushes and closes the file safely.

### Example Usage

```python
write_file(path="/path/to/myrepo/.git/refs/heads/master", data=b"abc123def456")
```

---

## `get_local_master_hash`

### Purpose

Retrieves the current commit hash of the local `master` branch.

### Parameters

- `repo_dir` (str): Path to the root of the Git repository.

### Description

- Reads the contents of `.git/refs/heads/master`.
- Returns the commit hash as a string.
- Returns `None` if no commit hash is found (e.g., empty repository).

### Example Usage

```python
current_master = get_local_master_hash(repo_dir="/path/to/myrepo")
print(f"Current master commit: {current_master}")
```

---

## `write_index`

### Purpose

Writes the current in-memory index entries back to the index file (`.git/index`), persisting staged changes.

### Parameters

- `repo_dir` (str): Path to the root Git repository.
- `index_entries` (list): A list of index entries representing staged files.

### Description

- Serializes the index entries according to Git index file format.
- Calculates and appends the checksum.
- Writes the serialized data atomically to `.git/index`.

### Example Usage

```python
write_index(repo_dir="/path/to/myrepo", index_entries=current_index_entries)
```

---

## ASCII Diagram: Commit Object Structure

```
+------------------------------+
|          Commit Object        |
+------------------------------+
| tree <tree_hash>             |----+
| parent <parent_hash> (opt)   |    |
| author <author_info>         |    |
| committer <committer_info>  |    |
|                             |    |
| <commit message>             |    |
+------------------------------+    |
                                    |
+------------------------------+    |
|          Tree Object          |<---+
+------------------------------+
| mode filename hash           |----+
| mode filename hash           |    |
| ...                         |    |
+------------------------------+
```

---

This documentation file provides a detailed guide to the commit mechanism in pygit, clarifying how core functions interact to record repository history. For related operations such as repository initialization, object management, and index handling, see the companion files in the **Core Repository Operations** and **Change Workflows** sections.