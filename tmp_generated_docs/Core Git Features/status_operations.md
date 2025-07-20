---
sidebar_position: 4
---

# Working Copy Status Determination in pygit

## Overview

This document explains how pygit determines the status of the working copy by comparing the Git index (staging area) to the working directory. It covers the main functions involved in this process, including `status()`, `get_status()`, and `read_index()`, as well as supporting helpers like `read_file()` and `hash_object()`. Understanding these functions is essential for grasping how pygit detects changes, identifies staged and unstaged modifications, and reports the current state of files relative to the repository. This file fits within the broader "Core Git Features" section of the documentation, complementing index operations and object handling.

---

## Function Documentation

### `status()`

**Purpose:**  
The `status()` function serves as the main entry point for determining the working copy status. It orchestrates the comparison between the index and the working directory to identify file changes such as modifications, additions, deletions, and untracked files.

**Parameters:**  
- `repo_path` (str): Path to the root of the Git repository.

**Operation:**  
1. Calls `read_index()` to load the current index entries into memory.  
2. Calls `get_status()` with the index data and working directory path.  
3. Collects and returns the status information for each relevant file.

**Example Usage:**

```python
repo_path = "/path/to/repo"
current_status = status(repo_path)
for filepath, state in current_status.items():
    print(f"{filepath}: {state}")
```

---

### `get_status(index_entries, working_dir)`

**Purpose:**  
`get_status()` compares the index entries with the files present in the working directory to detect differences indicating file status changes.

**Parameters:**  
- `index_entries` (list): List of index entries, each representing a file tracked by Git.  
- `working_dir` (str): Path to the working directory.

**Operation:**  
1. Iterates over each index entry.  
2. For each file, reads the current file content using `read_file()`.  
3. Computes the hash of the current file content using `hash_object()`.  
4. Compares the computed hash with the hash stored in the index entry.  
5. Categorizes the file status based on comparison results (e.g., modified, deleted, unchanged).  
6. Detects untracked files by comparing directory contents with index entries.

**Example Usage:**

```python
index_entries = read_index("/path/to/repo/.git/index")
working_dir = "/path/to/repo"
status_map = get_status(index_entries, working_dir)
print(status_map["README.md"])  # Outputs: "modified" or "unmodified"
```

---

### `read_index(index_path)`

**Purpose:**  
Reads and parses the Git index file, returning a structured list of index entries representing the currently staged files.

**Parameters:**  
- `index_path` (str): Full path to the Git index file (commonly `.git/index`).

**Operation:**  
1. Opens the index file in binary mode.  
2. Parses the header to validate and extract metadata.  
3. Iteratively reads each index entry, decoding fields such as file path, object hash, timestamps, and file mode.  
4. Returns a list of parsed index entries.

**Example Usage:**

```python
index_entries = read_index("/path/to/repo/.git/index")
for entry in index_entries:
    print(entry.path, entry.sha1)
```

---

### `read_file(file_path)`

**Purpose:**  
Reads the full content of a file from the working directory.

**Parameters:**  
- `file_path` (str): Path to the file to be read.

**Operation:**  
1. Opens the specified file in binary mode.  
2. Reads and returns the entire file content as bytes.

**Example Usage:**

```python
content = read_file("/path/to/repo/README.md")
print(content.decode("utf-8"))
```

---

### `hash_object(data, obj_type="blob")`

**Purpose:**  
Computes the Git object hash (SHA-1) for the given data, simulating how Git hashes objects in the repository.

**Parameters:**  
- `data` (bytes): Binary content of the object to hash.  
- `obj_type` (str, optional): Type of Git object (default is `"blob"`).

**Operation:**  
1. Prepares the object header with the format `{obj_type} {len(data)}\0`.  
2. Concatenates the header and data.  
3. Computes the SHA-1 hash of the concatenated bytes.  
4. Returns the hexadecimal string representation of the hash.

**Example Usage:**

```python
file_content = read_file("/path/to/repo/README.md")
file_hash = hash_object(file_content)
print(file_hash)  # Example: "3a7bd3e2360a7a8392b4987c8b0a8bacef5b6f1e"
```

---

## Status Determination Flow Diagram

This ASCII diagram illustrates the core process of determining file status by comparing the index and working directory:

```
+-----------------+          +----------------+          +-------------------+
|                 |          |                |          |                   |
|  Read Git Index +--------->+  Iterate Index +--------->+   For each file:   |
|  (.git/index)   |          |  Entries       |          |                   |
|                 |          |                |          |                   |
+-----------------+          +----------------+          +---------+---------+
                                                                  |
                                                                  v
                                                    +-----------------------------+
                                                    | Read current file content    |
                                                    | (read_file)                  |
                                                    +-----------------------------+
                                                                  |
                                                                  v
                                                    +-----------------------------+
                                                    | Compute file hash            |
                                                    | (hash_object)                |
                                                    +-----------------------------+
                                                                  |
                                                                  v
                                                    +-----------------------------+
                                                    | Compare with index hash      |
                                                    |                             |
                                                    | - If same: file unchanged    |
                                                    | - If different: modified     |
                                                    +-----------------------------+
                                                                  |
                                                                  v
                                                    +-----------------------------+
                                                    | Detect deleted/untracked     |
                                                    | files by directory scan      |
                                                    +-----------------------------+
                                                                  |
                                                                  v
                                                    +-----------------------------+
                                                    | Aggregate status results     |
                                                    +-----------------------------+
```

---

This documentation should serve as a comprehensive reference for developers looking to understand or extend the working copy status detection in pygit. For further details on related concepts such as index manipulation and object storage, see the [index operations](index_operations.md) and [object handling and cat_file](object_handling_and_cat_file.md) documentation files.