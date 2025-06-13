# repository.md

# Functions for Initializing Repositories and Checking Status

---

## Overview

The `repository.md` file documents key functions related to repository initialization and status checking within the `pygit` project. Positioned under the "Repository Initialization and Structure" section of the documentation tree, this file complements `init.md` by focusing on functions that manage the state of the repository post-initialization, such as committing changes, adding files, and inspecting the repository status.

These functions interact closely with the Git index, object store, and references, providing essential capabilities to create commits, track changes, and maintain the repository's integrity. This documentation serves developers working on or using `pygit` to understand how repository state is manipulated programmatically.

---

## Function Documentation

### 1. `commit(message, author=None)`

**Purpose:**  
Create a new commit object on the master branch representing the current state of the index, with a commit message and optional author information.

**Parameters:**  
- `message` (str): The commit message describing the changes.  
- `author` (str, optional): Author string in the format `"Name <email>"`. If not provided, environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL` are used.

**Operation:**  
1. Writes the current index as a tree object by calling `write_tree()`.  
2. Retrieves the current commit hash of the local master branch as the parent commit.  
3. Constructs the author and committer metadata with timestamps and timezones.  
4. Assembles commit object data including tree hash, parent hash (if any), author, committer, and message.  
5. Hashes and stores the commit object using `hash_object()`.  
6. Updates the `.git/refs/heads/master` reference with the new commit's SHA-1 hash.  
7. Prints confirmation of the commit and returns its SHA-1 hash.

**Example Usage:**

```python
sha1 = commit("Initial commit")
print(f"New commit created: {sha1}")
```

---

### 2. `get_status()`

**Purpose:**  
Inspect the working directory compared to the Git index to identify changed, new, and deleted files.

**Returns:**  
A tuple `(changed_paths, new_paths, deleted_paths)`, each a list of file paths as strings.

**Operation:**  
1. Walks the working directory recursively, ignoring the `.git` directory, collecting all file paths.  
2. Reads the Git index to obtain tracked file entries.  
3. Compares file contents by hashing current files and comparing with stored blob hashes in the index.  
4. Determines:  
   - `changed`: files present in both working directory and index but with modified content.  
   - `new`: files present in working directory but not in the index.  
   - `deleted`: files tracked in the index but missing in the working directory.

**Example Usage:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### 3. `add(paths)`

**Purpose:**  
Add specified files to the Git index, preparing them for commit.

**Parameters:**  
- `paths` (list of str): List of file paths to add.

**Operation:**  
1. Normalizes paths to use forward slashes.  
2. Reads current index entries and removes any entries matching the paths being added (to avoid duplicates).  
3. For each path:  
   - Reads file contents and hashes them as a blob object (`hash_object`).  
   - Gathers file metadata (ctime, mtime, mode, etc.) via `os.stat`.  
   - Creates a new `IndexEntry` with metadata and blob SHA-1.  
4. Sorts all entries by path and writes the updated index back using `write_index()`.

**Example Usage:**

```python
add(['src/main.py', 'README.md'])
print("Files added to index")
```

---

### 4. `status()`

**Purpose:**  
Display the status of the working copy by printing lists of changed, new, and deleted files.

**Operation:**  
1. Calls `get_status()` to retrieve file status categories.  
2. Prints each category with the corresponding file paths, if any exist.

**Example Usage:**

```python
status()
# Output:
# changed files:
#    src/main.py
# new files:
#    docs/usage.md
# deleted files:
#    old_script.py
```

---

### 5. `init(repo)`

**Purpose:**  
Initialize a new repository directory with its `.git` structure.

**Parameters:**  
- `repo` (str): Path to the repository directory to create.

**Operation:**  
1. Creates the main repository directory.  
2. Creates `.git` directory and subdirectories: `objects`, `refs`, and `refs/heads`.  
3. Writes the `HEAD` file pointing to the master branch ref.  
4. Prints confirmation message.

**Example Usage:**

```python
init('my_new_repo')
# Output:
# initialized empty repository: my_new_repo
```

---

### 6. `write_tree()`

**Purpose:**  
Write a tree object representing the current state of the index.

**Returns:**  
SHA-1 hash string of the newly created tree object.

**Operation:**  
1. Reads the index entries.  
2. For each entry, constructs a tree entry with mode and path, concatenated with the object's SHA-1.  
3. Hashes the concatenated tree entries as a tree object using `hash_object`.  
4. Returns the tree's SHA-1 hash.

**Constraints:**  
Currently supports only a flat directory structure (no nested paths).

**Example Usage:**

```python
tree_sha1 = write_tree()
print(f"Tree object created: {tree_sha1}")
```

---

### 7. `read_index()`

**Purpose:**  
Read the Git index file and parse it into a list of `IndexEntry` objects.

**Returns:**  
List of `IndexEntry` instances representing files tracked in the index.

**Operation:**  
1. Reads `.git/index` file content.  
2. Verifies checksum and index signature/version.  
3. Iteratively parses entries, extracting metadata and file paths.  
4. Assembles and returns a list of entries.

**Example Usage:**

```python
entries = read_index()
for entry in entries:
    print(f"{entry.path} - {entry.sha1.hex()}")
```

---

### 8. `write_index(entries)`

**Purpose:**  
Write a list of `IndexEntry` objects back to the Git index file.

**Parameters:**  
- `entries` (list of `IndexEntry`): Entries to write.

**Operation:**  
1. Packs each entry's metadata and path into binary format.  
2. Constructs the index file header with signature, version, and entry count.  
3. Calculates SHA-1 checksum of the index data and appends it.  
4. Writes the full content to `.git/index`.

**Example Usage:**

```python
write_index(entries)
print("Index updated")
```

---

### 9. `read_file(path)`

**Purpose:**  
Read the contents of a file as bytes.

**Parameters:**  
- `path` (str): Path to the file.

**Returns:**  
Byte content of the file.

**Example Usage:**

```python
data = read_file('README.md')
print(data.decode())
```

---

### 10. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute SHA-1 hash for Git object data of given type and optionally write it to the object store.

**Parameters:**  
- `data` (bytes): Raw object data.  
- `obj_type` (str): Type of Git object (`'blob'`, `'tree'`, `'commit'`, etc.).  
- `write` (bool): If `True`, write compressed object data to `.git/objects`.

**Returns:**  
SHA-1 hex string of the object.

**Example Usage:**

```python
sha1 = hash_object(b'Hello World\n', 'blob')
print(f"Blob object hash: {sha1}")
```

---

### 11. `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit hash of the local master branch.

**Returns:**  
SHA-1 hex string of the master commit or `None` if no commits exist.

**Example Usage:**

```python
master_hash = get_local_master_hash()
print(f"Local master commit: {master_hash}")
```

---

### 12. `ls_files(details=False)`

**Purpose:**  
List files currently in the Git index.

**Parameters:**  
- `details` (bool): If `True`, print file mode, SHA-1, stage number, and path. Otherwise, just paths.

**Example Usage:**

```python
ls_files(details=True)
# Output:
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0  README.md
```

---

### 13. `diff()`

**Purpose:**  
Show a unified diff of files changed between the index and working copy.

**Operation:**  
1. Obtains list of changed files via `get_status()`.  
2. For each changed file:  
   - Reads blob data from index.  
   - Reads working copy file contents.  
   - Uses Python's `difflib` to generate and print unified diff.

**Example Usage:**

```python
diff()
# Output:
# --- file.txt (index)
# +++ file.txt (working copy)
# @@ -1 +1 @@
# -old line
# +new line
```

---

### ASCII Diagram: Repository State and Commit Flow

```
Working Directory
       |
       v
    [git add]
       |
       v
     Index (staging area)
       |
       v
    [git commit]
       |
       v
    Commit Object
       |
       v
    refs/heads/master (branch ref)
```

This diagram illustrates the relationship between the working directory, index, commit objects, and branch references, showing how `add` and `commit` functions transition repository state.

---

## Summary

The functions documented here provide fundamental Git repository operations in `pygit`, enabling repository initialization, file tracking, status inspection, and commit creation. Together, they bridge the gap between the user's working directory and the Git object store, enforcing Git's data model and workflow programmatically.

This documentation complements other files in the "Repository Initialization and Structure" section by focusing on the operational aspects of repository state management after setup.