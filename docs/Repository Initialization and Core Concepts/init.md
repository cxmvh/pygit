# Repository Initialization and Core File Structure Setup (`init.md`)

---

## Overview

This document serves as a comprehensive guide and technical reference for initializing a Git repository within this project. It details the creation of the fundamental repository directory structure, setting up the `.git` directory, and managing essential core files such as `HEAD` and the object storage directories. This file is part of the broader "Repository Initialization and Core Concepts" section, which covers foundational Git operations including repository setup, commits, and index management.

Proper initialization is crucial as it lays the groundwork upon which version control commands operate, enabling the tracking of changes, commits, and history management. The main entry point for repository initialization is the `init()` function in the `pygit.init` module, which is responsible for creating the `.git` directory hierarchy and essential files.

---

## Function Documentation

### `init(repo)`

**Purpose:**  
Initialize a new Git repository in the specified directory `repo`. This involves creating the directory if it does not exist, setting up the `.git` directory, and populating it with the basic subdirectories and files needed to start tracking content.

**Parameters:**  
- `repo` (str): The file system path where the new repository directory will be created.

**Operation Steps:**  
1. Create the main repository directory at the path specified by `repo`.  
2. Inside this directory, create a `.git` directory to hold Git metadata.  
3. Within `.git`, create essential subdirectories:  
   - `objects` (stores Git object files such as blobs, trees, and commits)  
   - `refs` (stores references to commit hashes)  
   - `refs/heads` (stores references to branch heads)  
4. Create a `HEAD` file inside `.git` that points to the default branch reference: `refs/heads/master`.  
5. Print a confirmation message indicating successful initialization.

**Example Usage:**

```python
import pygit

# Initialize a new repository in the directory 'my_new_repo'
pygit.init('my_new_repo')

# Output:
# initialized empty repository: my_new_repo
```

**ASCII Diagram: Repository Directory Structure After Initialization**

```
my_new_repo/
└── .git/
    ├── HEAD                      # Points to refs/heads/master
    ├── objects/                  # Stores Git object files
    ├── refs/
    │   └── heads/                # Stores branch references
```

---

### `write_file(path, data)`

**Purpose:**  
Write raw byte data to a file at the specified path. This utility function is used throughout the repository initialization process to create and update files such as `HEAD` and objects in the `.git` directory.

**Parameters:**  
- `path` (str): File path where data should be written.  
- `data` (bytes): Byte content to write to the file.

**Operation Steps:**  
1. Open the file at `path` in binary write mode (`'wb'`).  
2. Write the byte data to the file.  
3. Close the file to ensure data is flushed to disk.

**Example Usage:**

```python
from pygit import write_file

# Write the string 'Hello, Git!' as bytes to a file
write_file('example.txt', b'Hello, Git!')
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash of Git object data of a given type (`blob`, `tree`, `commit`, etc.) and optionally write the compressed object data to the Git object store.

**Parameters:**  
- `data` (bytes): Raw content of the object.  
- `obj_type` (str): Git object type, e.g., `'blob'`, `'tree'`, or `'commit'`.  
- `write` (bool, optional): Whether to write the object to the `.git/objects` directory. Defaults to `True`.

**Operation Steps:**  
1. Construct the object header in the format: `<obj_type> <length>`, where length is the size of `data`.  
2. Concatenate the header, a null byte, and the data bytes to form the complete object content.  
3. Compute the SHA-1 hash of the complete object content.  
4. If `write` is `True`, compress the object content using zlib and write it to the `.git/objects/` directory using the first two hex characters of the SHA-1 as a subdirectory and the remaining characters as the filename.  
5. Return the SHA-1 hash as a hex string.

**Example Usage:**

```python
from pygit import hash_object

# Hash a blob object containing file data, and store it in the object database
blob_data = b'Hello, Git repository!'
sha1_hash = hash_object(blob_data, 'blob')

print(f'Object stored with SHA-1: {sha1_hash}')
```

---

### `write_index(entries)`

**Purpose:**  
Serialize and write a list of index entries to the Git index file (`.git/index`). The index tracks the current staging area for the repository.

**Parameters:**  
- `entries` (list of `IndexEntry`): List of entries representing files staged for commit.

**Operation Steps:**  
1. For each `IndexEntry`, pack its metadata fields into a binary structure according to Git's index file format.  
2. Append the file path and pad the entry to an 8-byte alignment.  
3. Concatenate all packed entries with a header containing the signature (`DIRC`), version number, and number of entries.  
4. Compute the SHA-1 checksum of the entire index data (excluding the last 20 bytes).  
5. Write the concatenated index data and checksum to `.git/index`.

**Example Usage:**

```python
from pygit import write_index, IndexEntry
import os

# Prepare index entries (example with a single file)
stat_result = os.stat('example.txt')
entry = IndexEntry(
    ctime_s=int(stat_result.st_ctime),
    ctime_n=0,
    mtime_s=int(stat_result.st_mtime),
    mtime_n=0,
    dev=stat_result.st_dev,
    ino=stat_result.st_ino,
    mode=stat_result.st_mode,
    uid=stat_result.st_uid,
    gid=stat_result.st_gid,
    size=stat_result.st_size,
    sha1=bytes.fromhex('...'),  # SHA-1 of the file content
    flags=len('example.txt'),
    path='example.txt'
)

write_index([entry])
```

---

### `commit(message, author=None)`

**Purpose:**  
Create a commit object representing the current state of the index, update the `master` branch reference to the new commit, and return the commit SHA-1 hash.

**Parameters:**  
- `message` (str): Commit message describing the changes.  
- `author` (str, optional): Author name and email in the format `"Name <email>"`. Defaults to environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Operation Steps:**  
1. Write the current index to a tree object and get its SHA-1 hash.  
2. Retrieve the current commit hash of the local `master` branch (if any).  
3. Compose commit metadata including tree hash, parent commit, author, committer, timestamp, and timezone.  
4. Create the commit object content by concatenating these fields along with the commit message.  
5. Hash the commit object and write it to the object store.  
6. Update the `refs/heads/master` reference to point to this new commit.  
7. Print a confirmation message with the commit hash and return the SHA-1.

**Example Usage:**

```python
from pygit import commit

commit_hash = commit("Initial commit")
print(f"Commit created with hash: {commit_hash}")
```

---

### `get_status()`

**Purpose:**  
Determine the status of the working copy by comparing the current files on disk with the Git index. Returns lists of changed, new, and deleted files relative to the index.

**Returns:**  
- Tuple of three lists: `(changed_paths, new_paths, deleted_paths)`

**Operation Steps:**  
1. Recursively walk the working directory excluding `.git/`, collecting all file paths.  
2. Read the current Git index entries and map by path.  
3. Identify changed files where the blob hash differs between the index and the working copy.  
4. Identify new files in the working directory not present in the index.  
5. Identify deleted files present in the index but missing from the working directory.  
6. Return sorted lists of these categorized paths.

**Example Usage:**

```python
from pygit import get_status

changed, new, deleted = get_status()

print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `add(paths)`

**Purpose:**  
Add one or more files to the Git index, preparing them to be committed.

**Parameters:**  
- `paths` (list of str): File paths to add to the index.

**Operation Steps:**  
1. Normalize file paths to use forward slashes.  
2. Read the current index entries and exclude any entries matching the specified paths.  
3. For each new path:  
   - Read the file contents and hash as a blob object.  
   - Retrieve file system metadata (stat).  
   - Create a new `IndexEntry` for the file.  
4. Append new entries to the index and sort by path.  
5. Write the updated index back to `.git/index`.

**Example Usage:**

```python
from pygit import add

# Add multiple files to the index
add(['README.md', 'src/main.py'])
```

---

### `write_tree()`

**Purpose:**  
Create a Git tree object from the current index entries representing the state of the working directory.

**Returns:**  
- SHA-1 hash string of the created tree object.

**Operation Steps:**  
1. Read all index entries.  
2. For each entry, construct a tree entry combining file mode, path, and SHA-1.  
3. Concatenate all tree entries and hash the combined data as a tree object.  
4. Return the SHA-1 of the new tree object.

**Example Usage:**

```python
from pygit import write_tree

tree_hash = write_tree()
print(f"Tree object created: {tree_hash}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit hash of the local `master` branch.

**Returns:**  
- SHA-1 commit hash string, or `None` if no commit exists.

**Operation Steps:**  
1. Read the contents of `.git/refs/heads/master`.  
2. Return the commit hash as a string or `None` if the file does not exist.

**Example Usage:**

```python
from pygit import get_local_master_hash

master_hash = get_local_master_hash()
print(f"Current master commit: {master_hash}")
```

---

## Summary Diagram: Repository Initialization Flow

```
+-----------------+
| init(repo)       |
|-----------------|
| - Create repo/   |
| - Create .git/   |
| - Create objects/|
| - Create refs/   |
| - Create HEAD    |
+-----------------+
         |
         v
+-----------------+
| write_file(path, |
| data)           |
+-----------------+

```

---

This documentation covers the core functions involved in repository initialization and the basic setup required to start using Git functionality within this project. It provides a foundation for further operations such as adding files, committing changes, and managing the repository state.