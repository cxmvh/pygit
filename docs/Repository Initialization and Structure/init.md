# Repository Initialization and .git Directory Setup

---

## Overview

The `init.md` documentation describes the process and functions involved in initializing a new Git repository within the `pygit` project. This includes creating the necessary directory structure, setting up the `.git` directory, and preparing the repository for subsequent Git operations. This file fits within the broader "Repository Initialization and Structure" section, providing foundational knowledge critical for understanding how the repository is prepared for version control. Proper initialization is essential for all higher-level Git commands such as commit, push, and status to function correctly.

---

## Important Functions

### 1. `init(repo)`

#### Purpose
Initializes a new Git repository by creating a directory named `repo` and setting up the `.git` directory structure inside it. This prepares the repository to track changes, store objects, and maintain references.

#### Parameters
- `repo` (str): The name/path of the directory where the repository is to be initialized.

#### Operation Steps
1. Creates the root repository directory.
2. Inside the repository, creates the `.git` directory.
3. Within `.git`, creates subdirectories:
   - `objects` — to store Git objects (blobs, trees, commits).
   - `refs` — for Git references.
   - `refs/heads` — specifically for branch heads.
4. Writes the initial `HEAD` file pointing to `refs/heads/master`, indicating the default branch.
5. Prints a confirmation message indicating successful initialization.

#### Example Usage
```python
import pygit

pygit.init('my_new_repo')
```
This will create a directory `my_new_repo` with the proper `.git` directory structure inside.

#### ASCII Diagram of `.git` Directory Structure After Initialization

```
my_new_repo/
└── .git/
    ├── objects/
    ├── refs/
    │   └── heads/
    └── HEAD  # Contains: ref: refs/heads/master
```

---

### 2. `write_file(path, data)`

#### Purpose
Writes raw byte data to a specified file path. Used internally to write configuration files and Git object files.

#### Parameters
- `path` (str): File path where data should be written.
- `data` (bytes): Data contents to write.

#### Operation Steps
1. Opens the file at `path` in binary write mode.
2. Writes the provided `data` bytes into the file.
3. Closes the file safely.

#### Example Usage
```python
pygit.write_file('.git/HEAD', b'ref: refs/heads/master')
```

---

### 3. `hash_object(data, obj_type, write=True)`

#### Purpose
Computes a SHA-1 hash of Git object data and optionally writes the compressed object to the Git object store.

#### Parameters
- `data` (bytes): The raw content of the Git object.
- `obj_type` (str): Type of Git object (`'blob'`, `'tree'`, `'commit'`, etc.).
- `write` (bool): If `True`, writes the object to the `.git/objects` directory.

#### Operation Steps
1. Constructs a header of the form `"<obj_type> <length>"`.
2. Concatenates the header, a null byte, and the data.
3. Computes the SHA-1 hash of the full content.
4. If `write` is `True`:
   - Determines path in the `.git/objects` directory based on SHA-1.
   - Compresses and writes the data if not already present.
5. Returns the SHA-1 hash as a hexadecimal string.

#### Example Usage
```python
data = b'example file content'
sha1 = pygit.hash_object(data, 'blob')
print(f'Object stored with SHA-1: {sha1}')
```

---

### 4. `write_index(entries)`

#### Purpose
Writes a list of Git index entries to the `.git/index` file. The index tracks the state of files staged for commit.

#### Parameters
- `entries` (list of `IndexEntry`): The list of index entries representing tracked files.

#### Operation Steps
1. Packs each `IndexEntry` into a binary format with metadata and path.
2. Builds a header with signature, version, and entry count.
3. Concatenates all packed entries.
4. Computes SHA-1 checksum for the entire index content.
5. Writes the full index data and checksum to `.git/index`.

#### Example Usage
```python
entries = read_index()
# Modify entries as needed
pygit.write_index(entries)
```

---

### 5. `commit(message, author=None)`

#### Purpose
Creates a new commit object representing the current state of the repository index, recording changes with a message and author.

#### Parameters
- `message` (str): Commit message describing the changes.
- `author` (str, optional): Author's identity. Defaults to environment variables if not provided.

#### Operation Steps
1. Writes the current index state to a tree object.
2. Retrieves the current commit hash of `master` as a parent.
3. Formats author and committer information with timestamps.
4. Constructs commit object content with tree, parent, author, committer, and message.
5. Hashes and writes the commit object to the object store.
6. Updates the `refs/heads/master` reference to point to the new commit.
7. Prints a confirmation with the commit hash.

#### Example Usage
```python
commit_hash = pygit.commit('Initial commit')
print(f'Created commit {commit_hash}')
```

---

### 6. `get_status()`

#### Purpose
Determines the status of the working directory by comparing tracked files in the index to the actual files on disk.

#### Returns
Tuple of three lists:
- `changed_paths`: Files modified since last commit.
- `new_paths`: New files not yet tracked.
- `deleted_paths`: Files removed from working directory.

#### Operation Steps
1. Recursively collects all file paths in the working directory excluding `.git`.
2. Reads current index entries and builds a path map.
3. Compares SHA-1 hashes of files on disk to those in the index.
4. Identifies changed, new, and deleted files.
5. Returns sorted lists of these paths.

#### Example Usage
```python
changed, new, deleted = pygit.get_status()
print('Changed:', changed)
print('New:', new)
print('Deleted:', deleted)
```

---

### 7. `add(paths)`

#### Purpose
Stages specified files by adding them to the Git index, computing their hashes, and updating index entries.

#### Parameters
- `paths` (list of str): Paths of files to add.

#### Operation Steps
1. Normalizes paths to use forward slashes.
2. Reads the existing index entries.
3. Removes entries matching the given paths to avoid duplicates.
4. For each path:
   - Reads file content.
   - Hashes it as a blob object.
   - Creates a new `IndexEntry` with file metadata.
5. Adds new entries and sorts them.
6. Writes updated index back to disk.

#### Example Usage
```python
pygit.add(['README.md', 'setup.py'])
```

---

### 8. `write_tree()`

#### Purpose
Creates a Git tree object representing the current state of the index, capturing file modes, names, and object hashes.

#### Returns
- SHA-1 hash string of the created tree object.

#### Operation Steps
1. Reads the index entries.
2. For each top-level file, formats an entry with mode and filename.
3. Concatenates all entries and hashes them as a tree object.
4. Returns the SHA-1 hash of the tree.

#### Example Usage
```python
tree_hash = pygit.write_tree()
print(f'Tree object created with hash: {tree_hash}')
```

---

### 9. `get_local_master_hash()`

#### Purpose
Retrieves the current commit hash of the local `master` branch from the reference file.

#### Returns
- SHA-1 hash string or `None` if no commit exists.

#### Operation Steps
1. Reads the contents of `.git/refs/heads/master`.
2. Returns the commit hash as a string.
3. Handles missing file gracefully by returning `None`.

#### Example Usage
```python
master_hash = pygit.get_local_master_hash()
print(f'Local master commit is {master_hash}')
```

---

### 10. `read_file(path)`

#### Purpose
Reads the entire contents of a file as bytes.

#### Parameters
- `path` (str): Path to the file.

#### Returns
- Contents of the file as bytes.

#### Example Usage
```python
content = pygit.read_file('README.md')
print(content.decode())
```

---

### 11. `read_index()`

#### Purpose
Reads and parses the Git index file, returning a list of `IndexEntry` objects representing tracked files.

#### Returns
- List of `IndexEntry` objects.

#### Operation Steps
1. Reads `.git/index`.
2. Verifies checksum and signature.
3. Parses each entry including metadata and path.
4. Returns the list of entries.
5. Returns empty list if index does not exist.

#### Example Usage
```python
entries = pygit.read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

### 12. `status()`

#### Purpose
Prints a human-readable status summary of the working directory changes.

#### Operation Steps
1. Calls `get_status()` to get changed, new, and deleted files.
2. Prints lists of files categorized by change type.

#### Example Usage
```python
pygit.status()
# Output:
# changed files:
#    README.md
# new files:
#    new_script.py
# deleted files:
#    old_file.txt
```

---

## Summary

This documentation covers the core initialization functionality of the `pygit` project. By understanding `init()` and its related helper functions such as `write_file()`, `hash_object()`, and `write_index()`, users and developers can grasp how a Git repository is bootstrapped from scratch. The `.git` directory structure is critical for storing all Git objects and references. This foundational knowledge enables effective use and further development of Git operations implemented in subsequent modules.

---

# Appendix: `.git` Directory Layout After Initialization

```
.git/
├── HEAD               # Points to current branch reference
├── objects/           # Stores all Git objects (blobs, trees, commits)
│   └── <object dirs>  # SHA-1 prefix folders
├── refs/
│   └── heads/         # Branch heads (e.g., master)
└── index              # Git index file (created later after adding files)
```

---

This concludes the documentation for repository initialization and `.git` directory setup.