# commit.md

## Overview

The `commit.md` file documents the mechanisms for creating commit objects, locating commit-related Git objects, and updating local branches within the repository. This file resides within the **Tree and Commit Management** section of the overall documentation, which focuses on handling Git trees and commits. The functions detailed here are crucial for recording repository history by committing staged changes, traversing commit and tree objects, and managing branch references, especially the local `master` branch. These operations form the backbone of version control workflows, enabling users to capture snapshots of their project state and maintain a coherent commit history.

---

## Function Documentation

---

### `commit(message, author=None)`

#### Purpose
Creates a new commit object representing the current state of the staging area (index) and updates the local `master` branch to point to this new commit. This function encapsulates the commit metadata, including parent commits, author information, timestamps, and the commit message.

#### Parameters
- `message` (str): The commit message describing the changes.
- `author` (str, optional): The author string in the format `"Name <email>"`. If not provided, it defaults to the environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

#### Operation Steps
1. Calls `write_tree()` to create a tree object representing the current index.
2. Retrieves the current commit hash of the local `master` branch using `get_local_master_hash()`, to be set as the parent commit.
3. If no author is provided, constructs the author string from the environment.
4. Records the current timestamp and timezone offset.
5. Builds the commit object content with lines specifying:
   - The tree object hash.
   - The parent commit hash (if any).
   - The author and committer information with timestamps.
   - The commit message.
6. Hashes this commit content with `hash_object()` to create and store the commit object.
7. Updates the `.git/refs/heads/master` file with the new commit hash.
8. Prints confirmation and returns the commit SHA-1 hash.

#### Usage Example
```python
commit_hash = commit("Fix bug in data processing module")
print(f"Created commit {commit_hash}")
```

---

### `get_local_master_hash()`

#### Purpose
Fetches the current commit hash that the local `master` branch points to. Returns `None` if no commit exists yet.

#### Returns
- `str` or `None`: The SHA-1 hex string of the current commit on `master` or `None` if the branch is uninitialized.

#### Operation Steps
- Reads the `.git/refs/heads/master` file.
- Returns its contents as a string after stripping whitespace.
- Handles the case where the file does not exist gracefully by returning `None`.

#### Usage Example
```python
current_commit = get_local_master_hash()
if current_commit:
    print(f"Current master commit: {current_commit}")
else:
    print("No commits found on master branch yet.")
```

---

### `write_tree()`

#### Purpose
Creates a Git tree object that represents the state of the current index (staging area). This tree object contains entries for files staged in the index.

#### Returns
- `str`: The SHA-1 hex string of the newly created tree object.

#### Preconditions
- The index must be properly populated with entries representing files.

#### Operation Steps
1. Reads the index entries via `read_index()`.
2. For each entry:
   - Asserts that the path is a top-level file (no subdirectories supported currently).
   - Constructs a tree entry composed of the file mode, path, a null byte, and the SHA-1 hash of the file blob.
3. Concatenates all tree entries into a single byte string.
4. Calls `hash_object()` with type `'tree'` to store this tree object.
5. Returns the tree’s SHA-1 hash.

#### Usage Example
```python
tree_hash = write_tree()
print(f"Created tree object: {tree_hash}")
```

---

### `find_commit_objects(commit_sha1)`

#### Purpose
Recursively gathers the full set of Git object hashes referenced by a commit, including the commit itself, its associated tree, and all parent commits and their trees.

#### Parameters
- `commit_sha1` (str): The SHA-1 hash of the commit to analyze.

#### Returns
- `set` of `str`: Set of SHA-1 hashes of all related objects.

#### Operation Steps
1. Adds the commit SHA-1 to the set.
2. Reads the commit object using `read_object()`.
3. Extracts the tree hash from the commit.
4. Calls `find_tree_objects(tree)` to get all objects under the tree.
5. Extracts parent commit hashes and recursively adds their objects.
6. Returns the complete set.

#### Usage Example
```python
commit_objects = find_commit_objects('a1b2c3d4...')
print(f"Objects reachable from commit: {commit_objects}")
```

---

### `find_tree_objects(tree_sha1)`

#### Purpose
Recursively collects all object hashes within a Git tree, including blobs and nested subtrees, plus the tree itself.

#### Parameters
- `tree_sha1` (str): The SHA-1 hash of the tree object.

#### Returns
- `set` of `str`: Set of SHA-1 hashes of all objects contained in the tree.

#### Operation Steps
1. Includes the tree SHA-1 itself.
2. Reads the tree entries using `read_tree()`.
3. For each entry:
   - If a directory (subtree), recursively calls itself.
   - Otherwise, adds the blob SHA-1.
4. Returns the collected set.

#### Usage Example
```python
tree_objects = find_tree_objects('d4e5f6a7...')
print(f"Objects in tree: {tree_objects}")
```

---

### `read_object(sha1_prefix)`

#### Purpose
Retrieves a Git object given a SHA-1 prefix, returning its type and raw data.

#### Parameters
- `sha1_prefix` (str): Prefix of the SHA-1 hash to identify the object.

#### Returns
- Tuple `(obj_type, data)`:
  - `obj_type` (str): One of `'blob'`, `'tree'`, `'commit'`, etc.
  - `data` (bytes): Raw decompressed object data.

#### Operation Steps
1. Uses `find_object()` to get the object file path.
2. Reads and decompresses the object file bytes.
3. Parses the header for object type and size.
4. Extracts the object data.
5. Validates size and returns type and data.

#### Usage Example
```python
obj_type, data = read_object('a1b2c3')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### `write_file(path, data)`

#### Purpose
Writes raw data bytes to a file at the specified path.

#### Parameters
- `path` (str): File path to write to.
- `data` (bytes): Data to write.

#### Usage Example
```python
write_file('.git/refs/heads/master', b'a1b2c3d4e5f6\n')
```

---

### ASCII Diagram: Commit Creation Workflow

```
+--------------+       +--------------+       +------------------+
|  Index State | ----> |  write_tree  | ----> |  Tree Object Hash |
+--------------+       +--------------+       +------------------+
                                                      |
                                                      v
                                              +------------------+
                                              | get_local_master  |
                                              |  (parent commit)  |
                                              +------------------+
                                                      |
                                                      v
                                              +------------------+
                                              |   Prepare commit  |
                                              |    object data    |
                                              +------------------+
                                                      |
                                                      v
                                              +------------------+
                                              |   hash_object    |
                                              | (store commit obj)|
                                              +------------------+
                                                      |
                                                      v
                                              +------------------+
                                              | Update master ref |
                                              +------------------+
```

---

### `push(git_url, username=None, password=None)`

#### Purpose
Pushes the local `master` branch commits to a remote Git repository.

#### Parameters
- `git_url` (str): URL of the remote Git repository.
- `username` (str, optional): Username for authentication; defaults to environment variable `GIT_USERNAME`.
- `password` (str, optional): Password for authentication; defaults to environment variable `GIT_PASSWORD`.

#### Operation Steps
1. Retrieves the remote `master` commit hash using `get_remote_master_hash()`.
2. Gets the local `master` commit hash with `get_local_master_hash()`.
3. Identifies missing objects on the remote with `find_missing_objects()`.
4. Prints an update message with commit hashes and object count.
5. Builds Git protocol lines and packfile data including missing objects.
6. Sends data to remote using `http_request()`.
7. Parses and verifies server response.
8. Returns tuple `(remote_sha1, missing)`.

#### Usage Example
```python
remote, missing = push('https://github.com/user/repo.git')
print(f"Pushed to remote: {remote}, objects sent: {len(missing)}")
```

---

## Summary

This documentation covers the essential functions for creating commits, traversing commit and tree objects, and updating the local branch references. The commit process starts from the index, writes a tree object, creates a commit object linking to the current `master` commit (parent), and updates the branch reference. The ability to find all objects reachable from a commit facilitates operations like pushing changes to a remote repository. The file completes the core Git commit lifecycle within the `pygit` implementation.

For detailed usage of other related functions like `read_index`, `hash_object`, or `find_object`, please refer to their respective documentation files within the **Object Handling** and **Tree and Commit Management** sections.