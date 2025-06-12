# commit.md

## Overview

This document details the processes involved in committing changes to a Git repository using the pygit implementation. It covers creating commit objects, writing tree objects from the index, updating the master branch reference, and related utilities for managing commits. Situated within the "Commit and Branch Management" section of the repository, this file plays a crucial role in explaining how changes tracked in the index are finalized into commit snapshots, enabling version history tracking and branch management. The documented functions form the core of creating new commits, linking to parent commits, and managing the master branch reference within the `.git` directory.

---

## Function Documentation

---

### `commit(message, author=None)`

**Purpose:**  
Create a new commit object representing the current state of the index and update the local `master` branch to point to this commit.

**Parameters:**  
- `message` (str): The commit message describing the changes.  
- `author` (str, optional): The author string in the format `"Name <email>"`. If not provided, the environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL` are used.

**Operation Steps:**  
1. Call `write_tree()` to write the current index contents as a tree object and obtain its SHA-1 hash.  
2. Retrieve the latest commit hash from the local `master` branch using `get_local_master_hash()` to set as the parent commit (if any).  
3. Construct the author and committer metadata including timestamp and timezone offset.  
4. Prepare the commit content lines:
   - `tree <tree_sha1>`
   - `parent <parent_sha1>` (if a parent exists)
   - `author <author> <timestamp> <timezone>`
   - `committer <author> <timestamp> <timezone>`
   - A blank line
   - Commit message
   - A blank line  
5. Join and encode the commit content, then hash it as a `commit` object via `hash_object`.  
6. Write the resulting commit SHA-1 to the `refs/heads/master` file to update the branch pointer.  
7. Print a confirmation message and return the commit SHA-1.

**Example Usage:**
```python
commit_hash = commit("Add initial project files")
print(f"New commit created: {commit_hash}")
```

---

### `write_tree()`

**Purpose:**  
Generate a Git tree object from the current index entries representing the snapshot of the working directory.

**Parameters:**  
None.

**Operation Steps:**  
1. Read the current index entries using `read_index()`.  
2. For each index entry:
   - Ensure the path is top-level (no subdirectories) for this implementation.  
   - Format the entry as `<mode> <filename>\0<sha1>` (mode as octal string, null byte as separator).  
3. Concatenate all entries and hash the combined data as a `tree` object using `hash_object()`.  
4. Return the SHA-1 hash of the tree object.

**Example Usage:**
```python
tree_hash = write_tree()
print(f"Tree object created with SHA-1: {tree_hash}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit hash pointed to by the local `master` branch.

**Parameters:**  
None.

**Operation Steps:**  
1. Read the contents of `.git/refs/heads/master`.  
2. If the file does not exist (e.g., no commits yet), return `None`.  
3. Otherwise, decode and strip whitespace to return the commit SHA-1 string.

**Example Usage:**
```python
current_master = get_local_master_hash()
if current_master:
    print(f"Current master commit: {current_master}")
else:
    print("No commits on master branch yet.")
```

---

### `write_file(path, data)`

**Purpose:**  
Write binary data to a file at the specified path.

**Parameters:**  
- `path` (str): File system path where data should be written.  
- `data` (bytes): Data to write.

**Operation Steps:**  
1. Open the file in binary write mode.  
2. Write the given data bytes to the file.  
3. Close the file.

**Example Usage:**
```python
write_file('.git/refs/heads/master', b'abc1234...\n')
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash for given object data of a specified type, optionally writing the compressed object to the Git object store.

**Parameters:**  
- `data` (bytes): Raw data of the object.  
- `obj_type` (str): Git object type (e.g., `'commit'`, `'tree'`, `'blob'`).  
- `write` (bool): Whether to write the compressed object to the `.git/objects` directory.

**Operation Steps:**  
1. Construct the object header as `"{obj_type} {len(data)}"`.  
2. Concatenate header, null byte, and data.  
3. Compute SHA-1 hash of the concatenated bytes.  
4. If `write` is `True`, check if the object file exists in `.git/objects/XX/XXXX...`, create directories if needed, and write the compressed data using `write_file()`.  
5. Return the SHA-1 hash as a hex string.

**Example Usage:**
```python
blob_hash = hash_object(b'Hello, World!\n', 'blob')
print(f"Blob stored with SHA-1: {blob_hash}")
```

---

### `write_index(entries)`

**Purpose:**  
Serialize and write a list of index entries to the `.git/index` file.

**Parameters:**  
- `entries` (list of `IndexEntry`): Index entries representing files staged for commit.

**Operation Steps:**  
1. For each index entry, pack the metadata fields and path with appropriate padding to align to 8 bytes.  
2. Concatenate all packed entries.  
3. Construct the index header with signature `DIRC`, version 2, and number of entries.  
4. Compute SHA-1 checksum of header and entries.  
5. Write the combined data and checksum to `.git/index` via `write_file()`.

**Example Usage:**
```python
entries = read_index()
write_index(entries)
```

---

### `get_status()`

**Purpose:**  
Detect differences between the working directory and the index, returning changed, new, and deleted file paths.

**Parameters:**  
None.

**Operation Steps:**  
1. Walk the working directory recursively, ignoring `.git` directory, collecting all file paths.  
2. Read index entries and map paths to entries.  
3. Identify changed files as those present in both but with differing blob SHA-1 hashes.  
4. Identify new files as those in the working directory but not in the index.  
5. Identify deleted files as those in the index but no longer in the working directory.  
6. Return sorted tuples of `(changed, new, deleted)` paths.

**Example Usage:**
```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `add(paths)`

**Purpose:**  
Add specified files to the Git index, updating or creating index entries for each path.

**Parameters:**  
- `paths` (list of str): File paths to add to the index.

**Operation Steps:**  
1. Normalize paths to use forward slashes.  
2. Read current index entries, excluding any with the given paths to avoid duplicates.  
3. For each path:
   - Read file contents and create a blob object via `hash_object()`.  
   - Collect file metadata (mode, size, timestamps, etc.).  
   - Create a new `IndexEntry` with this data.  
4. Append new entries and sort all entries by path.  
5. Write the updated entries back to the index with `write_index()`.

**Example Usage:**
```python
add(['README.md', 'src/main.py'])
```

---

### `read_index()`

**Purpose:**  
Read and parse the Git index file, returning a list of `IndexEntry` objects representing staged files.

**Parameters:**  
None.

**Operation Steps:**  
1. Attempt to read `.git/index`; return empty list if missing.  
2. Verify SHA-1 checksum of index data to ensure integrity.  
3. Parse header including signature and version.  
4. Iterate over entries:
   - Unpack fixed metadata fields.  
   - Extract file path string.  
   - Create `IndexEntry` objects.  
5. Return list of entries.

**Example Usage:**
```python
entries = read_index()
for e in entries:
    print(e.path, e.sha1.hex())
```

---

### ASCII Diagram — Commit Object Structure

```
Commit Object Content:

+----------------+       +-------------------+       +------------------+
| tree <tree_sha1>| ----> | parent <parent_sha1>| ---> | parent <parent_sha1> (optional)
+----------------+       +-------------------+       +------------------+
        |
        v
+-------------------------------------+
| author <author> <timestamp> <tz>    |
+-------------------------------------+
| committer <author> <timestamp> <tz>|
+-------------------------------------+

Message:

<blank line>
<commit message>
<blank line>
```

This structure is serialized as text lines and hashed as the commit object.

---

### `push(git_url, username=None, password=None)`

**Purpose:**  
Push the local `master` branch commits and associated objects to a remote Git repository.

**Parameters:**  
- `git_url` (str): URL of the remote Git repository.  
- `username` (str, optional): Username for authentication; defaults to environment variable `GIT_USERNAME`.  
- `password` (str, optional): Password for authentication; defaults to environment variable `GIT_PASSWORD`.

**Operation Steps:**  
1. Retrieve the remote master commit hash via `get_remote_master_hash()`.  
2. Retrieve the local master commit hash via `get_local_master_hash()`.  
3. Identify missing objects on the remote using `find_missing_objects()`.  
4. Prepare the update command line indicating refs to update.  
5. Create a packfile with missing objects using `create_pack()`.  
6. Send an authenticated HTTP request to remote's `git-receive-pack` endpoint with update command and packfile data.  
7. Parse server response and verify success messages (`unpack ok`, `ok refs/heads/master`).  
8. Return remote SHA-1 and set of missing objects pushed.

**Example Usage:**
```python
remote_url = "https://example.com/user/repo.git"
push(remote_url)
print("Push completed successfully.")
```

---

## Summary

This documentation provides a comprehensive reference for the key functions responsible for committing changes within the pygit project. The interplay between reading and writing the index, creating tree and commit objects, and updating branch references is essential for maintaining a coherent Git history. The included `push` function extends commit management by enabling synchronization with remote repositories. The modular design ensures clarity and extensibility, supporting further development of Git functionality.

For further details on related areas such as object storage, index management, and remote operations, please consult the corresponding documentation files within the repository structure.