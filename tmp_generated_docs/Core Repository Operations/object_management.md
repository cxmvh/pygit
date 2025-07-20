---
sidebar_position: 3
---

# Git Object Management

This document provides comprehensive reference material on Git object storage, retrieval, and display within the repository. It details the mechanisms and functions responsible for hashing, reading, locating, and writing Git objects such as blobs, trees, and commits. This file is part of the broader **Core Repository Operations** section, which consolidates foundational Git internals including repository initialization, object management, and index handling. Understanding object management is essential for grasping how Git tracks content efficiently and reconstructs repository state.

---

## cat_file

### Purpose

The `cat_file` function emulates the behavior of the Git command `git cat-file`. It reads and displays Git objects stored in the object database. This function supports multiple modes to show object data in different formats, such as raw content, type, or size.

### Parameters

- **object_hash** (`str`): The SHA-1 or SHA-256 hash identifying the Git object.
- **mode** (`str`): Determines the display format. Common modes include:
  - `"blob"`: Show the raw content of a blob object.
  - `"type"`: Display the type of the object (blob, tree, commit, tag).
  - `"size"`: Show the size of the object in bytes.
  - `"pretty"`: Display a human-readable form, e.g., formatted commit message or tree listing.

### Operation

1. Locate the object file in the `.git/objects` directory using the given hash.
2. Read and decompress the stored object data.
3. Parse the object header to determine its type and size.
4. Depending on the mode:
   - Return raw content (for blobs).
   - Return the object type string.
   - Return the object size.
   - Format and pretty-print the object (e.g., decoding commit messages or tree entries).
5. Handle errors if the object is missing or corrupted.

### Example

```python
# Display the content of a blob object
cat_file("4a1b2c3d4e5f6g7h8i9j0klmnopqrstuvwx1234", mode="blob")

# Show the type of a commit object
cat_file("9f8e7d6c5b4a3a2b1c0d9e8f7a6b5c4d3e2f1a0b", mode="type")

# Pretty-print a tree object
cat_file("abcdef1234567890abcdef1234567890abcdef12", mode="pretty")
```

---

## read_object

### Purpose

The `read_object` function retrieves a Git object by its hash and returns a structured representation of its contents. This serves as a lower-level utility used by other functions to interpret Git objects beyond raw data display.

### Parameters

- **object_hash** (`str`): The identifier hash of the object to read.

### Operation

1. Use the object hash to locate and decompress the stored object.
2. Parse the header to extract object type and size.
3. Deserialize the payload based on the object type:
   - For blobs: return raw bytes.
   - For trees: parse entries into a list of `(mode, filename, hash)` tuples.
   - For commits: parse commit metadata like tree hash, parent commits, author, committer, and message.
4. Return an object or dictionary encapsulating the parsed information.

### Example

```python
obj = read_object("9f8e7d6c5b4a3a2b1c0d9e8f7a6b5c4d3e2f1a0b")
if obj.type == "commit":
    print("Commit message:", obj.message)
elif obj.type == "tree":
    for mode, fname, hash in obj.entries:
        print(f"{mode} {fname} {hash}")
```

---

## read_tree

### Purpose

The `read_tree` function processes a Git tree object, extracting its entries to reconstruct the directory structure at a given commit or tree state.

### Parameters

- **tree_hash** (`str`): Hash identifying the tree object to read.

### Operation

1. Read the tree object using `read_object`.
2. For each entry in the tree:
   - Extract the mode (file permissions), filename, and object hash.
   - Classify entries as blobs (files) or trees (subdirectories).
3. Optionally, recursively read nested trees to reconstruct full directory hierarchies.

### Example

```python
entries = read_tree("abcdef1234567890abcdef1234567890abcdef12")
for mode, filename, obj_hash in entries:
    print(f"{mode} {filename} {obj_hash}")
```

---

## find_object

### Purpose

`find_object` locates a Git object by a partial or full hash prefix within the object database. This enables abbreviated hash lookup similar to Git’s own behavior.

### Parameters

- **prefix** (`str`): The initial characters of the object hash.

### Operation

1. Search the `.git/objects` directory for objects whose hash starts with the prefix.
2. If multiple matches are found, raise an ambiguity error.
3. Return the full hash of the matched object if unique.
4. Return `None` or raise an error if no matching object is found.

### Example

```python
full_hash = find_object("4a1b2c")
print("Full object hash:", full_hash)
```

---

## Modes for Displaying Object Data

Git objects can be displayed in various modes to suit different use cases. These modes are supported primarily by `cat_file`:

- **blob**: Outputs raw file content.
- **type**: Shows the object type (`blob`, `tree`, `commit`, `tag`).
- **size**: Displays the size of the object content.
- **pretty**: Formats the object for human-friendly reading, such as showing commit details or tree listings.

---

## Processing Trees and Commits

Git trees and commits form the backbone of repository state tracking:

- **Trees** represent directory snapshots, containing entries with file modes, filenames, and hashes to blobs or other trees.
- **Commits** reference a tree object and zero or more parent commits, along with metadata like author, committer, timestamps, and commit messages.

### Commit Structure (simplified)

```
commit <object size>\0
tree <tree hash>
parent <parent commit hash> (optional)
author <author info>
committer <committer info>

<commit message>
```

### ASCII Diagram: Git Object Relationships

```
[Commit Object]
     |
     v
[Tree Object] -----> [Blob Object (file)]
     |
     +----> [Blob Object (file)]
     |
     +----> [Tree Object (subdirectory)] -----> [Blob Object]
```

This hierarchy enables Git to efficiently store and retrieve snapshots of the entire repository at each commit.

---

This documentation serves as a technical reference for developers working with Git internals or implementing Git-compatible tooling. For further details on repository initialization and index management, see the respective files in the **Core Repository Operations** section.