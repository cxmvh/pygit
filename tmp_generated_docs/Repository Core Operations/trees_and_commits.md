---
sidebar_position: 4
---

# Trees and Commits

## Overview

This document provides a unified reference for handling tree and commit objects within the repository. It explains the key functions involved in reading and writing tree objects (`read_tree`, `write_tree`), creating commits, and traversing both tree and commit objects. The documentation clarifies how these operations relate to the Git index and the object store, highlighting their roles in core repository workflows such as staging files and recording snapshots. This file complements other core operation documentation in the repository, linking the index management and object storage mechanisms with the representation of project states in commit history.

---

## read_tree

### Purpose

`read_tree` is responsible for reading a tree object from the object store and converting it into an in-memory representation that reflects the directory structure and contents at a specific point in the repository’s history. This function is crucial for reconstructing the state of the working directory or index from a commit or tree object.

### Parameters

- **tree_sha**: The SHA-1 hash identifier of the tree object to read.
- **object_store**: Interface or path to the object database where the tree object is stored.

### Operation

1. Locate the tree object in the object store using the provided SHA.
2. Decompress and parse the raw tree data, which consists of entries describing filenames, file modes, and SHA hashes for blobs or subtrees.
3. Construct an in-memory tree structure mapping file paths to their corresponding object hashes and modes.
4. Return this structured representation, suitable for indexing or working directory updates.

### Example Usage

```python
tree_sha = "a1b2c3d4e5f6g7h8i9j0klmnopqrstuvwx"
tree = read_tree(tree_sha, object_store)
print(tree)
# Output:
# {
#   'README.md': {'mode': '100644', 'sha': 'f1e2d3c4b5a697887766554433221100ffeeddaa'},
#   'src': {
#       'mode': '040000',
#       'sha': '1234567890abcdef1234567890abcdef12345678',
#       'entries': { ... }
#   }
# }
```

---

## write_tree

### Purpose

`write_tree` writes the current state of a directory tree or index entries back into the object store as a new tree object. This function encodes the directory structure and file references into the Git internal tree format and returns the SHA of the newly created tree object.

### Parameters

- **tree_dict**: An in-memory representation of the tree structure, mapping filenames to their modes and object SHAs.
- **object_store**: Interface or path to the object database for storing the new tree object.

### Operation

1. Serialize the tree dictionary into the Git tree object format:
   - For each entry, write mode, filename, and SHA in the prescribed format.
2. Compress and write the serialized data to the object store.
3. Compute and return the SHA-1 hash of the newly written tree object.

### Example Usage

```python
tree_dict = {
    'README.md': {'mode': '100644', 'sha': 'f1e2d3c4b5a697887766554433221100ffeeddaa'},
    'src': {
        'mode': '040000',
        'sha': '1234567890abcdef1234567890abcdef12345678',
        'entries': { 
            # nested entries
        }
    }
}
new_tree_sha = write_tree(tree_dict, object_store)
print(f"Created tree object with SHA: {new_tree_sha}")
```

---

## commit Creation

### Purpose

The commit creation process involves constructing a commit object that records a snapshot of the repository at a given state. It references a tree object that captures the project files and includes metadata such as author, committer, message, and parent commits. This function encapsulates the logic to create, write, and store commit objects.

### Parameters

- **tree_sha**: SHA of the tree object representing the snapshot.
- **parent_shas**: List of SHA hashes of parent commits (empty for initial commit).
- **author**: Author metadata (name, email, timestamp).
- **committer**: Committer metadata (often same as author).
- **message**: Commit message describing the changes.
- **object_store**: Interface to write the commit object.

### Operation

1. Format the commit object content, including:
   - Tree reference (`tree <sha>`)
   - Parent references (`parent <sha>`) for each parent commit
   - Author and committer metadata lines
   - Commit message
2. Write the formatted commit content to the object store as a commit object.
3. Return the SHA of the newly created commit.

### Example Usage

```python
commit_sha = create_commit(
    tree_sha=new_tree_sha,
    parent_shas=["abcdef1234567890abcdef1234567890abcdef12"],
    author={'name': 'Alice', 'email': 'alice@example.com', 'timestamp': 1686000000},
    committer={'name': 'Alice', 'email': 'alice@example.com', 'timestamp': 1686000100},
    message="Add new feature implementation",
    object_store=object_store
)
print(f"Created commit with SHA: {commit_sha}")
```

---

## Traversing Tree and Commit Objects

### Purpose

Traversal functions allow inspection and navigation through the hierarchical tree structures and linked commit objects. This is essential for operations such as generating diffs, checking out files, and analyzing project history.

### Tree Traversal

- Recursively visit tree entries.
- For each tree entry:
  - If it is a blob, process the file object.
  - If it is a subtree, recurse into it.

### Commit Traversal

- Start from a commit object.
- Access its tree SHA and parent commits.
- Optionally traverse parents to walk commit history.

### ASCII Diagram: Commit and Tree Relationship

```
Commit Object (commit_sha)
    |
    |-- references Tree Object (tree_sha)
           |
           |-- entries (files and subtrees)
                   |
                   |-- blob objects (file contents)
                   |-- subtree objects (nested directories)
    |
    |-- references Parent Commit(s)
```

### Example Usage: Tree Traversal

```python
def traverse_tree(tree_sha, object_store):
    tree = read_tree(tree_sha, object_store)
    for name, entry in tree.items():
        if entry['mode'] == '040000':  # subtree
            print(f"Entering directory: {name}")
            traverse_tree(entry['sha'], object_store)
        else:
            print(f"File: {name} (sha: {entry['sha']})")

# Start traversal from a commit's tree
commit = read_commit(commit_sha, object_store)
traverse_tree(commit['tree'], object_store)
```

---

## Relationship to Index and Object Store

- The **index** holds a staged snapshot of the working directory, recording file paths, modes, and blob SHAs.
- `read_tree` and `write_tree` translate between the index (or working directory) and tree objects stored in the object database.
- Commit objects reference tree objects, encapsulating the entire snapshot of the project state.
- These functions enable core operations like staging changes, committing snapshots, and checking out historical states by linking index contents, tree objects, and commits.

---

This documentation integrates the core concepts and functions related to trees and commits, providing a foundation for understanding how Git represents and manipulates project states internally. For more details on index management, see [index_and_status.md](index_and_status.md), and for object storage internals, refer to [object_handling_and_storage.md](object_handling_and_storage.md).