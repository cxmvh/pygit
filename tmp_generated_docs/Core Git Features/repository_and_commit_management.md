---
sidebar_position: 1
---

# Repository And Commit Management

## Overview

This file documents the core functionalities related to repository initialization and commit management within the pygit implementation. It covers how a Git repository is initialized (`pygit.init`), how objects are hashed, read, and written into the object store, the process of writing tree objects that represent directory states, managing the Git index (staging area), and finally how commits are created and branches updated (`pygit.commit`). This documentation situates itself within the broader "Core Git Features" section that addresses essential Git operations implemented in pygit, providing developers with a detailed understanding of foundational Git mechanics.

---

## `init`

### Purpose
The `init` function initializes a new Git repository in a specified directory. It sets up the necessary repository structure, including directories and configuration files, making the directory ready for version control operations.

### Parameters
- **path** (`str`): The filesystem path where the new Git repository should be initialized.

### Description
`init` creates a `.git` directory inside the provided path, along with essential subdirectories like `objects` and `refs`. It also creates initial files such as `HEAD` to point to the default branch (usually `refs/heads/master` or `refs/heads/main`). This function ensures the repository is in a valid state to start tracking files and commits.

### Operational Steps
1. Verify if the path exists and is writable.
2. Create the `.git` directory inside the path.
3. Inside `.git`, create the `objects` directory to store Git objects.
4. Create the `refs/heads` directory to store branch references.
5. Write the `HEAD` file pointing to the default branch reference.
6. Optionally, initialize configuration files as needed.

### Example Usage

```python
import pygit

# Initialize a new Git repository in the 'my_project' directory
pygit.init('my_project')
```

---

## `hash_object`

### Purpose
Hashes file contents or data blobs and writes them to the Git object store, returning the SHA-1 hash of the stored object.

### Parameters
- **data** (`bytes`): Raw data to be hashed and stored.
- **obj_type** (`str`): The type of Git object, e.g., `'blob'`, `'tree'`, `'commit'`.
- **write** (`bool`): If `True`, the object is written to the store; if `False`, only the hash is computed.

### Description
This function computes the SHA-1 hash of a Git object, which consists of a header and the data. The header is formatted as `<type> <size>\0`. When `write=True`, it compresses the object and saves it under the `.git/objects` directory following Git's object storage format.

### Operational Steps
1. Construct the object header: `"{obj_type} {len(data)}\0"`.
2. Concatenate the header and the data.
3. Compute SHA-1 hash of the concatenated bytes.
4. If `write` is `True`, compress the object and write it to the objects directory, stored under the first two characters of the hash as a directory, and the remaining characters as the filename.
5. Return the SHA-1 hash as a hexadecimal string.

### Example Usage

```python
data = b"Hello World\n"
sha = pygit.hash_object(data, obj_type='blob', write=True)
print(f"Object stored with SHA-1: {sha}")
```

---

## `write_tree`

### Purpose
Creates a tree object representing the state of a directory by recursively hashing files and subdirectories and writing the tree object to the object store.

### Parameters
- **directory_path** (`str`): Path to the directory to be written as a tree.
- **write** (`bool`): Whether to write the tree object to the object store.

### Description
`write_tree` collects all entries in a directory, including subdirectories and files, and creates a tree object. Each entry contains mode, filename, and SHA-1 hash of the corresponding object (blob for files, tree for subdirectories). The tree object is then hashed and optionally written to the object store.

### Operational Steps
1. List entries in the target directory.
2. For each file entry:
   - Hash the file content as a blob object.
   - Record the mode, filename, and blob's SHA-1.
3. For each directory entry:
   - Recursively call `write_tree` to get the SHA-1 of subtrees.
4. Construct the tree object content by concatenating entries in the format:  
   `"<mode> <filename>\0<sha_bytes>"`
5. Hash the tree object, optionally write it, and return its SHA-1.

### Example Usage

```python
tree_sha = pygit.write_tree('my_project', write=True)
print(f"Tree object created with SHA-1: {tree_sha}")
```

### ASCII Diagram: Tree Object Structure

```
tree object:
+---------------------------+
| mode filename \0 sha (20) |
| mode filename \0 sha (20) |
|           ...             |
+---------------------------+
```

---

## `read_object`

### Purpose
Reads and decompresses a Git object from the object store by its SHA-1 hash, returning the object type and data.

### Parameters
- **sha** (`str`): SHA-1 hash of the object to read.

### Description
This function locates the object file in `.git/objects`, decompresses it, and splits it into the header and data. The header provides the object type and size, and the data is the actual content.

### Operational Steps
1. Locate the object file using the SHA-1 hash.
2. Decompress the object file contents using zlib.
3. Parse the header to extract the object type and size.
4. Return the object type and the raw data.

### Example Usage

```python
obj_type, data = pygit.read_object('e69de29bb2d1d6434b8b29ae775ad8c2e48c5391')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

## `update_index`

### Purpose
Manages the Git index (staging area) by adding, updating, or removing entries, reflecting changes before committing.

### Parameters
- **index_entries** (`list`): List of index entries representing files staged for commit.

### Description
`update_index` modifies the index file to reflect the current staging state. Each index entry contains file metadata, SHA-1 of the blob, and other attributes. This function writes the updated index entries to the index file in the proper format, including recalculating checksums.

### Operational Steps
1. Serialize index entries into the Git index file format.
2. Calculate a checksum for the entire index content.
3. Write the serialized content and checksum to the `.git/index` file.

### Example Usage

```python
entries = [
    # hypothetical index entry dictionary/list format
    {'path': 'README.md', 'sha': '...', 'mode': 0o100644, ...},
    # more entries
]
pygit.update_index(entries)
```

---

## `commit`

### Purpose
Creates a new commit object, writes it to the object store, and updates the current branch reference to point to the new commit.

### Parameters
- **message** (`str`): Commit message describing the changes.
- **author** (`str`): The author's name and email.
- **committer** (`str`): The committer's name and email.
- **tree_sha** (`str`): SHA-1 of the tree object representing the project state.
- **parent_sha** (`str` or `None`): SHA-1 of the parent commit (if any).

### Description
The `commit` function constructs a commit object containing metadata (author, committer, message), references the tree object representing the project state, and links to parent commits. It hashes and writes this commit object, then updates the branch reference (e.g., `refs/heads/master`) to point to the new commit SHA-1, effectively recording a new state in the repository history.

### Operational Steps
1. Construct the commit object contents, including:
   - Tree SHA-1 line
   - Parent SHA-1 line (if applicable)
   - Author and committer metadata with timestamps
   - Commit message
2. Hash and write the commit object to the object store.
3. Update the current branch reference file to point to the new commit SHA-1.

### Example Usage

```python
commit_sha = pygit.commit(
    message="Initial commit",
    author="Alice <alice@example.com>",
    committer="Alice <alice@example.com>",
    tree_sha=tree_sha,
    parent_sha=None
)
print(f"Created commit {commit_sha}")
```

### ASCII Diagram: Commit Object Structure

```
commit object:
+--------------------------+
| tree <tree_sha>          |
| parent <parent_sha>      |  (optional)
| author <author info>     |
| committer <committer>    |
|                          |
| <commit message>         |
+--------------------------+
```

---

## Summary Diagram: Repository Commit Flow

```
+----------------+
| pygit.init()   |  -- Initializes repository structure
+----------------+
         |
         v
+----------------+
| hash_object()  |  -- Hashes blobs, trees, commits
+----------------+
         |
         v
+----------------+
| write_tree()   |  -- Creates tree objects recursively
+----------------+
         |
         v
+----------------+
| update_index() |  -- Updates staging area
+----------------+
         |
         v
+----------------+
| commit()       |  -- Creates commit and updates branch
+----------------+
```

---

This documentation provides a comprehensive guide to the foundational pygit operations for repository initialization and commit management. For deeper details on object reading and index operations, please refer to the related documentation files in the [Core Git Features](./) section.