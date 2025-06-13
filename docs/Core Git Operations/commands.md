# commands.md

## Overview

This document provides detailed implementation information for core Git commands: `add`, `commit`, `status`, `diff`, and `push`. These commands form the backbone of typical Git workflows, enabling users to stage changes, record snapshots of the project, inspect the state of the working directory, review differences, and synchronize with remote repositories. Situated within the *Core Git Operations* section of the documentation tree, this file bridges repository initialization and index management with user-facing operations, illustrating how the underlying Git data structures and functions are orchestrated to implement these commands.

---

## Function Documentation

### `add(paths)`

**Purpose:**  
Stages specified files by adding their current content to the Git index, preparing them for the next commit.

**Parameters:**  
- `paths` (list of str): List of file paths to add to the index.

**Operation:**  
1. Normalize file paths to use forward slashes.  
2. Read the current index entries, filtering out any entries corresponding to the given paths to avoid duplicates.  
3. For each path:  
   - Read the file content and hash it as a blob object, storing it in the object store.  
   - Collect file metadata (ctime, mtime, device, inode, mode, uid, gid, size).  
   - Create a new `IndexEntry` with the metadata and blob SHA-1.  
4. Append new entries and sort the combined entry list by path.  
5. Write the updated index back to disk.

**Example Usage:**
```python
# Stage 'file1.txt' and 'src/app.py' for commit
add(['file1.txt', 'src/app.py'])
```

---

### `commit(message, author=None)`

**Purpose:**  
Creates a new commit object representing the current state of the index, recording the given commit message and author details.

**Parameters:**  
- `message` (str): Commit message describing the changes.  
- `author` (str, optional): Author string in the format `"Name <email>"`. Defaults to environment variables.

**Operation:**  
1. Write a tree object from the current index entries.  
2. Retrieve the current commit hash on `master` as the parent commit (if any).  
3. Format author and committer information with timestamps and timezone offsets.  
4. Construct the commit object content with tree, parent, author, committer, and message fields.  
5. Hash and store the commit object.  
6. Update the `refs/heads/master` reference with the new commit hash.  
7. Print a confirmation message and return the commit hash.

**Example Usage:**
```python
commit_hash = commit("Add initial project files")
print(f"Created commit {commit_hash}")
```

---

### `status()`

**Purpose:**  
Displays the current status of the working directory relative to the index, showing changed, new, and deleted files.

**Parameters:**  
- None

**Operation:**  
1. Retrieve sets of changed, new, and deleted paths using `get_status()`.  
2. Print lists of changed files, new files, and deleted files, grouped separately.

**Example Usage:**
```python
status()
# Output:
# changed files:
#    file1.txt
# new files:
#    new_script.py
# deleted files:
#    old_file.txt
```

---

### `diff()`

**Purpose:**  
Shows the content differences between the working directory and the index for files that have changed.

**Parameters:**  
- None

**Operation:**  
1. Use `get_status()` to get changed files.  
2. For each changed file:  
   - Read the blob object from the index and the working copy file content.  
   - Use `difflib.unified_diff` to generate a unified diff.  
   - Print the diff output, separating multiple files with a line of dashes.

**Example Usage:**
```python
diff()
# Output:
# --- file1.txt (index)
# +++ file1.txt (working copy)
# @@ -1,3 +1,4 @@
# -Line 1
# +Line 1 modified
# ...
```

---

### `push(git_url, username=None, password=None)`

**Purpose:**  
Pushes the local `master` branch commits and objects to a remote Git repository.

**Parameters:**  
- `git_url` (str): URL of the remote Git repository.  
- `username` (str, optional): Username for authentication; defaults to environment variable.  
- `password` (str, optional): Password for authentication; defaults to environment variable.

**Operation:**  
1. Retrieve the remote master commit hash.  
2. Retrieve the local master commit hash.  
3. Identify missing objects on the remote by comparing commit ancestry.  
4. Build protocol lines and create a packfile containing missing objects.  
5. Send a `git-receive-pack` HTTP request with authentication and data.  
6. Parse response lines to ensure successful unpack and update.  
7. Return a tuple of the remote SHA-1 before push and the set of pushed objects.

**Example Usage:**
```python
remote_sha1, missing_objects = push("https://example.com/myrepo.git")
print(f"Pushed {len(missing_objects)} objects to remote, previous SHA1: {remote_sha1}")
```

---

## Supporting Functions Overview

The above commands rely on several lower-level functions managing Git internals:

- **`hash_object(data, obj_type, write=True)`**: Hashes data as a Git object and optionally writes it to the object store.  
- **`read_index()` / `write_index(entries)`**: Read and write the Git index file containing staged file metadata.  
- **`write_tree()`**: Creates a tree object from current index entries representing the project snapshot.  
- **`get_local_master_hash()`**: Retrieves the current commit hash of the local master branch.  
- **`get_status()`**: Compares working directory files with the index to find changed, new, and deleted files.  
- **`read_object(sha1_prefix)`**: Reads a Git object by SHA-1 prefix, returning its type and data.

---

## ASCII Diagram: Command Interaction with Git Data Structures

```
Working Directory
    |
    | (add)
    v
Git Index -------------------\
    |                         \
    | (write_tree)             \
    v                          \
Git Object Store               Commit Object
    |                            |
    |                            | (commit)
    |                            v
    |----------------------> refs/heads/master (branch ref)
                                  |
                                  | (push)
                                  v
                            Remote Repository
```

- `add` stages files from the working directory into the Git index (staging area).  
- `commit` records the current index as a commit object referencing a tree and parent commits and updates the branch ref.  
- `push` uploads commits and objects from the local repository to a remote repository.

---

# End of `commands.md` documentation file content.