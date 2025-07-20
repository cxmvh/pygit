---
sidebar_position: 2
---

# Object Handling and Cat File Operations

## Overview

This document provides an in-depth explanation of how pygit reads, locates, and processes Git objects using their SHA-1 hashes. It covers the mechanisms for reading objects from the Git object store, decompressing their contents, parsing the raw data, and displaying the object data in various modes such as commit, tree, blob, size, type, and pretty format. This functionality corresponds primarily to the `pygit.cat_file` operation and is essential for inspecting and manipulating the underlying Git object database within pygit. As part of the "Core Git Features" section, this document complements other files handling repository initialization, commits, index operations, and status checks by focusing on the low-level object data access and presentation.

---

## Key Functions in Object Handling and Display

### `cat_file(sha, mode='blob', pretty=False)`

#### Purpose

The `cat_file` function is the main entry point for reading and displaying Git objects identified by their SHA-1 hash. It supports multiple modes to control how the object is processed and presented:

- **blob**: Display raw blob content.
- **commit**: Parse and display commit object details.
- **tree**: Parse and display tree object entries.
- **size**: Show the size of the object content.
- **type**: Display the object type (blob, tree, commit).
- **pretty**: Provide a human-readable, formatted output depending on object type.

#### Parameters

- `sha` (str): The SHA-1 hash string identifying the Git object.
- `mode` (str, optional): The mode for reading/displaying the object. Defaults to `'blob'`.
- `pretty` (bool, optional): Whether to pretty-print the output. Defaults to `False`.

#### Operation

1. **Locate object in the store**: Given the SHA-1 hash, determine the file path in the `.git/objects` directory based on the first two characters forming a subdirectory and the remaining characters as the filename.

2. **Read and decompress**: Open the object file and decompress it using zlib to retrieve the raw Git object data.

3. **Parse header**: Git objects have a header of the form `<type> <size>\0` followed by the object content. Extract the object type and size, verifying integrity.

4. **Process content by mode**:
   - If mode is `size`, return the size of the content.
   - If mode is `type`, return the object type.
   - If mode is `blob`, return the raw blob data.
   - If mode is `commit`, parse the commit fields (tree, parent(s), author, committer, message).
   - If mode is `tree`, parse tree entries (mode, filename, SHA).
   - If mode is `pretty`, parse and format the object content depending on its type for easy human reading.

5. **Return or display the processed output**.

#### Example Usage

```python
# Display the raw content of a blob object
blob_content = cat_file("a3f5c6e2b9d4f5a1234567890abcdef123456789")
print(blob_content)

# Show the size of a commit object
commit_size = cat_file("b7d2e1a94f8c7d1234567890abcdef1234567890", mode='size')
print(f"Commit size: {commit_size} bytes")

# Pretty print a commit object
cat_file("b7d2e1a94f8c7d1234567890abcdef1234567890", mode='commit', pretty=True)
```

---

### Supporting Function: `read_object(sha)`

#### Purpose

Reads and decompresses a Git object from the object store by its SHA-1 hash and returns the raw data including the header.

#### Parameters

- `sha` (str): SHA-1 hash of the object.

#### Operation

- Constructs the object file path from the SHA-1.
- Reads the compressed object file.
- Decompresses the content using zlib.
- Returns the decompressed raw data.

#### Example Usage

```python
raw_data = read_object("a3f5c6e2b9d4f5a1234567890abcdef123456789")
print(raw_data)
```

---

### Supporting Function: `parse_commit(raw)`

#### Purpose

Parses raw commit object data into a structured dictionary containing commit metadata and message.

#### Parameters

- `raw` (bytes): Raw commit object data (excluding Git header).

#### Operation

- Splits the raw data into header lines and message.
- Extracts commit fields such as:
  - `tree`: SHA of the tree object.
  - `parent`: SHA(s) of parent commit(s).
  - `author`: Author metadata line.
  - `committer`: Committer metadata line.
- Extracts the commit message.
- Returns a dictionary with parsed fields.

#### Example Usage

```python
commit_data = parse_commit(raw_commit_data)
print(f"Tree: {commit_data['tree']}")
print(f"Author: {commit_data['author']}")
print(f"Message:\n{commit_data['message']}")
```

---

### Supporting Function: `parse_tree(raw)`

#### Purpose

Parses raw tree object data and returns a list of entries, each representing a file or directory in the tree.

#### Parameters

- `raw` (bytes): Raw tree object data (excluding Git header).

#### Operation

- Iterates over the raw data entries.
- Extracts for each entry:
  - File mode (e.g., 100644).
  - Filename.
  - SHA-1 hash of the entry object.
- Returns a list of dictionaries representing tree entries.

#### Example Usage

```python
tree_entries = parse_tree(raw_tree_data)
for entry in tree_entries:
    print(f"{entry['mode']} {entry['sha']} {entry['filename']}")
```

---

## ASCII Diagram: Git Object Storage Layout

```text
.git/
└── objects/
    ├── a3/
    │   └── f5c6e2b9d4f5a1234567890abcdef123456789  # Object file for SHA a3f5c6...
    └── b7/
        └── d2e1a94f8c7d1234567890abcdef1234567890  # Object file for SHA b7d2e1...
```

- The first two characters of the SHA-1 form the directory name.
- The remaining 38 characters form the object filename.
- Files are compressed with zlib.

---

## Summary

This documentation file describes how pygit implements the reading and inspection of Git objects by SHA-1. By understanding the `cat_file` operation and its supporting parsing functions, developers can effectively inspect Git data structures such as commits, trees, and blobs in raw or human-friendly formats. This capability is fundamental to many Git operations and allows for flexible manipulation and debugging of Git repositories within pygit.