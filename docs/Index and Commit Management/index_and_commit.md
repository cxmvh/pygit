```mdx
# Git Index and Commit Management

This document provides an in-depth technical reference for managing the Git index, staging files, writing tree objects, and creating commits within a Git repository. It details the internal structure and operation of the Git index file, the process of adding files to the index, writing tree objects from the index state, and creating and storing commit objects. Additionally, it explains the `IndexEntry` data structure and the role of checksums in ensuring index integrity.

This file sits within the broader **Index and Commit Management** section of the documentation tree, complementing related topics such as repository initialization, object management, and working copy status. It is essential reading for developers working on Git internals or building Git-compatible tools that manipulate repository state programmatically.

---

## Git Index File Structure and Operations

The Git index, often stored in `.git/index`, is a binary file that tracks the staged state of the working directory. It acts as a staging area, representing a snapshot of the content to be committed.

### Index File Structure Overview

The index file contains:

- A fixed header including a signature (`"DIRC"`), version number, and the count of index entries.
- A series of **IndexEntry** records, each representing a staged file.
- A trailing checksum (SHA-1 hash) validating the index contents.

Each **IndexEntry** holds metadata about a file such as its path, timestamps, file mode, object ID (blob SHA-1), and flags.

```
+-------------------+
| Header            |
| - Signature       | "DIRC"
| - Version         | typically 2 or 3
| - Entry count     |
+-------------------+
| IndexEntry #1     |
+-------------------+
| IndexEntry #2     |
+-------------------+
| ...               |
+-------------------+
| Checksum (SHA-1)  |
+-------------------+
```

---

## IndexEntry Structure

An `IndexEntry` encapsulates the details of one file staged in the index:

| Field           | Description                                   |
|-----------------|-----------------------------------------------|
| `ctime`         | Creation time of the file (seconds + nanoseconds) |
| `mtime`         | Modification time of the file (seconds + nanoseconds) |
| `dev`           | Device number                                 |
| `ino`           | Inode number                                  |
| `mode`          | File mode bits (permissions and type)        |
| `uid`           | User ID of the file owner                     |
| `gid`           | Group ID of the file owner                     |
| `size`          | File size in bytes                            |
| `sha1`          | SHA-1 hash of the file content (blob object ID) |
| `flags`         | Flags including path length and stage         |
| `path`          | File path relative to repository root        |

---

## Key Functions

### `read_index(path: str) -> List[IndexEntry]`

Reads the Git index file from the given path and returns a list of `IndexEntry` objects.

**Parameters:**

- `path`: Filesystem path to the `.git/index` file.

**Returns:**

- List of parsed `IndexEntry` instances representing the current staged files.

**Operation:**

1. Open the index file in binary mode.
2. Validate the header signature and version.
3. Read the number of entries.
4. Parse each entry according to its binary layout.
5. Verify the trailing checksum matches the file contents.

**Example:**

```python
entries = read_index('.git/index')
for entry in entries:
    print(f"Staged file: {entry.path} ({entry.sha1.hex()})")
```

---

### `write_index(path: str, entries: List[IndexEntry]) -> None`

Writes the given list of `IndexEntry` objects back to the index file at `path`, updating the staging area.

**Parameters:**

- `path`: Target path for the index file (typically `.git/index`).
- `entries`: List of `IndexEntry` objects to write.

**Operation:**

1. Construct the index header with signature, version, and entry count.
2. Serialize each `IndexEntry` in the proper binary format.
3. Compute the checksum (SHA-1) of all previous bytes.
4. Append the checksum to the end of the file.
5. Write the constructed bytes atomically to avoid corruption.

**Example:**

```python
write_index('.git/index', updated_entries)
```

---

### `add_to_index(path: str, file_paths: List[str]) -> None`

Adds or updates files in the index based on the current working directory contents.

**Parameters:**

- `path`: Path to the `.git/index` file.
- `file_paths`: List of file paths relative to the repository root to stage.

**Operation:**

1. Read the existing index entries.
2. For each file path:
   - Read file metadata and content.
   - Compute the blob SHA-1 object ID.
   - Create or update the corresponding `IndexEntry`.
3. Write the updated list of entries back to the index.

**Example:**

```python
add_to_index('.git/index', ['README.md', 'src/main.py'])
```

---

### `write_tree(entries: List[IndexEntry], object_store_path: str) -> str`

Generates a tree object from the current index entries and stores it in the object database.

**Parameters:**

- `entries`: List of `IndexEntry` objects representing the staged files.
- `object_store_path`: Path to the `.git/objects` directory.

**Returns:**

- SHA-1 object ID of the newly created tree object.

**Operation:**

1. Organize entries by directory hierarchy.
2. Serialize tree entries in the format: `[mode] [filename]\0[SHA-1]`.
3. Write the tree object to the object store.
4. Return the tree object's SHA-1 hash.

**Example:**

```python
tree_sha1 = write_tree(entries, '.git/objects')
print(f"Created tree object {tree_sha1.hex()}")
```

---

### `create_commit(tree_sha1: str, parent_sha1s: List[str], author: str, message: str, object_store_path: str) -> str`

Creates a commit object referencing the given tree and parents, then stores it.

**Parameters:**

- `tree_sha1`: SHA-1 hash of the tree object representing the snapshot.
- `parent_sha1s`: List of parent commit SHA-1 hashes (empty for initial commit).
- `author`: Author information string (e.g., `"Alice <alice@example.com>"`).
- `message`: Commit message text.
- `object_store_path`: Path to the `.git/objects` directory.

**Returns:**

- SHA-1 object ID of the newly created commit.

**Operation:**

1. Construct commit object content including tree, parents, author, committer, and message fields.
2. Compute SHA-1 of the commit content.
3. Store the commit object in the object store.
4. Return the commit SHA-1.

**Example:**

```python
commit_sha1 = create_commit(tree_sha1, [], "Alice <alice@example.com>", "Initial commit", ".git/objects")
print(f"Created commit {commit_sha1.hex()}")
```

---

## Checksum Validation

The index file ends with a 20-byte SHA-1 checksum computed over all previous bytes of the file. This checksum ensures integrity and detects file corruption or incomplete writes.

When reading the index, your implementation should:

- Compute the SHA-1 hash of the entire file except the trailing checksum.
- Compare it against the stored checksum.
- Raise an error or warning if they do not match.

---

## ASCII Diagram: Index to Commit Flow

```
Working Directory
      |
      v
+-----------------+
| Add files to    |   read file metadata & content
| Index (staging) | -------------------->
+-----------------+                       |
      |                                  v
      |                      +-----------------+
      |                      | Git Index File  |
      |                      | (.git/index)    |
      |                      +-----------------+
      |                               |
      | write_index(entries)          |
      |------------------------------>|
      |                               |
      |                               v
      |                      +-----------------+
      |                      | Write Tree      |
      |                      +-----------------+
      |                               |
      |                      create tree object
      |                               |
      |------------------------------>|
      |                               |
      |                               v
      |                      +-----------------+
      |                      | Create Commit   |
      |                      +-----------------+
      |                               |
      |                      store commit object
      |                               |
      |------------------------------>|
      |                               v
      +----------------------> Commit SHA-1
```

---

This document outlines the core mechanisms underpinning Git's index and commit creation. Together with related documents such as [repository_initialization.md](../Repository%20Initialization%20and%20Structure/repository_initialization.md) and [object_management.md](../Git%20Object%20Management/object_management.md), it forms a comprehensive guide to Git's internal data handling.

```