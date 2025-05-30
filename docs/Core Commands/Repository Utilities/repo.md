# Repository Initialization and Status Reporting (`repo.md`)

---

## Overview

This document covers the core functionalities related to initializing a Git repository and reporting its status within the `pygit` project. It resides under the "Repository Utilities" section of the documentation tree and complements other modules like repository setup (`init.md`) and working copy management (`status.md`). The key roles of this file include creating the `.git` directory structure for a new repository, managing the index and commit processes, and providing clear status reports about the working copy relative to the repository state. These utilities are foundational for establishing and maintaining the repository’s internal structure and for user interactions with repository state information.

---

## Important Functions

### 1. `init(repo)`

**Purpose:**  
Initialize a new Git repository by creating the directory structure and essential files under the specified `repo` directory.

**Parameters:**  
- `repo` (str): The path where the new repository directory will be created.

**Operation:**  
- Creates the main repository directory and the `.git` subdirectory.  
- Within `.git`, creates necessary subdirectories: `objects`, `refs`, and `refs/heads`.  
- Writes the `HEAD` file to point to the default branch (`refs/heads/master`).  
- Prints a confirmation message upon successful initialization.

**Example Usage:**
```python
init('my-new-repo')
# Output:
# initialized empty repository: my-new-repo
```

**ASCII Diagram:**

```
my-new-repo/
└── .git/
    ├── objects/
    ├── refs/
    │   └── heads/
    └── HEAD  # contains 'ref: refs/heads/master'
```

---

### 2. `get_status()`

**Purpose:**  
Analyze the working directory and index to report which files have changed, which are new, and which have been deleted.

**Returns:**  
A tuple of three lists:  
- `changed_paths` (list of str): Files modified since the last commit.  
- `new_paths` (list of str): Files present in the working directory but not in the index.  
- `deleted_paths` (list of str): Files tracked in the index but missing from the working directory.

**Operation:**  
- Walks the working directory (excluding `.git`) to collect current file paths.  
- Reads the index entries to get tracked files and their blob hashes.  
- Compares the hashes of files in the working directory against the index to detect changes.  
- Identifies new files (in working directory but not in index).  
- Identifies deleted files (in index but not in working directory).

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
- Converts Windows-style paths to Unix-style.  
- Reads existing index entries and filters out entries corresponding to files being added.  
- For each new file:  
  - Reads file content and creates a blob object, storing its SHA-1 hash.  
  - Retrieves file metadata (timestamps, mode, ownership).  
  - Creates a new index entry with this information.  
- Sorts all entries by path.  
- Writes updated entries back to the index file.

**Example Usage:**
```python
add(['README.md', 'src/main.py'])
```

---

### 4. `commit(message, author=None)`

**Purpose:**  
Create a new commit object in the repository representing the current state of the index, with an associated commit message and author information.

**Parameters:**  
- `message` (str): Commit message describing changes.  
- `author` (str, optional): Author string in the format `"Name <email>"`. If omitted, environment variables are used.

**Returns:**  
- SHA-1 hash (str) of the created commit object.

**Operation:**  
- Calls `write_tree()` to create a tree object representing the index state.  
- Retrieves the current commit hash of `master` branch as parent (if any).  
- Formats author and committer information with timestamps and timezone offsets.  
- Constructs commit object content including tree, parent, author, committer, and message.  
- Hashes and stores the commit object.  
- Updates the `refs/heads/master` to point to the new commit.  
- Prints a confirmation message with the commit SHA-1.

**Example Usage:**
```python
sha1 = commit("Initial commit")
print(f"Created commit {sha1}")
```

---

### 5. `status()`

**Purpose:**  
Display the current status of the working directory with respect to the index, including lists of changed, new, and deleted files.

**Operation:**  
- Calls `get_status()` to retrieve file status lists.  
- Prints categorized lists of changed, new, and deleted files, if any.

**Example Usage:**
```python
status()
# Output example:
# changed files:
#     README.md
# new files:
#     src/utils.py
# deleted files:
#     old_script.py
```

---

### 6. `ls_files(details=False)`

**Purpose:**  
List files currently tracked in the Git index.

**Parameters:**  
- `details` (bool): If `True`, include detailed information (file mode, SHA-1 hash, and stage number).

**Operation:**  
- Reads index entries and prints file paths.  
- If detailed output is requested, prints mode, SHA-1, stage, and path.

**Example Usage:**
```python
ls_files()
# Output:
# README.md
# src/main.py

ls_files(details=True)
# Output:
# 100644 3b18e5d3... 0    README.md
# 100644 4a7e3bf2... 0    src/main.py
```

---

### 7. `write_tree()`

**Purpose:**  
Create a tree object representing the current index state and write it to the object store.

**Returns:**  
- SHA-1 hash (str) of the created tree object.

**Operation:**  
- Iterates over index entries, preparing tree entries with mode, path, and SHA-1.  
- Concatenates all tree entries into a single byte string.  
- Hashes and writes the tree object using `hash_object`.

**Example Usage:**
```python
tree_sha1 = write_tree()
print(f"Created tree object {tree_sha1}")
```

---

### 8. `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit hash of the local `master` branch.

**Returns:**  
- SHA-1 hash (str) of the latest commit on `master`, or `None` if none exists.

**Operation:**  
- Reads the contents of `.git/refs/heads/master`.  
- Returns the commit hash as a string or `None` if the file is missing.

**Example Usage:**
```python
current_commit = get_local_master_hash()
print(f"Current master commit: {current_commit}")
```

---

### 9. `read_index()`

**Purpose:**  
Read the Git index file and parse its entries into a list of `IndexEntry` objects.

**Returns:**  
- List of `IndexEntry` objects representing the current index state.

**Operation:**  
- Opens `.git/index` and reads its contents.  
- Validates checksum and file signature/version.  
- Parses each index entry, decoding file metadata and path.  
- Returns the list of entries.

**Example Usage:**
```python
entries = read_index()
for e in entries:
    print(f"{e.path} - mode: {oct(e.mode)}")
```

---

### 10. `write_index(entries)`

**Purpose:**  
Write a list of `IndexEntry` objects to the Git index file, updating the repository index.

**Parameters:**  
- `entries` (list of `IndexEntry`): The index entries to write.

**Operation:**  
- Packs each entry's metadata and path according to Git index format.  
- Computes and appends SHA-1 checksum of the index data.  
- Writes the data to `.git/index`.

**Example Usage:**
```python
write_index(entries)
```

---

### 11. `read_object(sha1_prefix)`

**Purpose:**  
Retrieve a Git object by its SHA-1 prefix and return its type and data.

**Parameters:**  
- `sha1_prefix` (str): Prefix of the SHA-1 hash identifying the object.

**Returns:**  
- Tuple `(object_type, data_bytes)` where `object_type` is a string such as `'blob'`, `'tree'`, or `'commit'`, and `data_bytes` holds the raw object data.

**Operation:**  
- Locates the object file in `.git/objects` based on SHA-1 prefix.  
- Decompresses and parses object header and content.  
- Validates size and returns parsed data.

**Example Usage:**
```python
obj_type, data = read_object('3b18e5d3')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### 12. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash of an object of a given type and optionally write it to the object store.

**Parameters:**  
- `data` (bytes): Raw data of the object.  
- `obj_type` (str): Type of the object (e.g., `'blob'`, `'tree'`, `'commit'`).  
- `write` (bool): Whether to write the object to disk (default `True`).

**Returns:**  
- SHA-1 hash (str) of the object.

**Operation:**  
- Prepares object header and concatenates with data.  
- Computes SHA-1 hash of full object.  
- If `write` is `True`, compresses and writes the object to `.git/objects`.  
- Returns the hash.

**Example Usage:**
```python
sha1 = hash_object(b'Hello world\n', 'blob')
print(f"Created blob object {sha1}")
```

---

### 13. `write_file(path, data)`

**Purpose:**  
Write raw bytes to a file at a specified path.

**Parameters:**  
- `path` (str): File path.  
- `data` (bytes): Data to write.

**Operation:**  
- Opens the file in binary write mode and writes the data.

**Example Usage:**
```python
write_file('.git/description', b'This is a test repository\n')
```

---

### 14. `status()`

**Purpose:**  
Print a human-readable status report of the working directory.

**Operation:**  
- Uses `get_status()` to identify changed, new, and deleted files.  
- Prints files grouped by their status category.

**Example Usage:**
```python
status()
```

---

## Additional Notes

- The commit process depends on writing a tree object that represents the current index snapshot.
- Status reporting relies on comparing hashes of working directory files versus index entries.
- Index reading and writing adhere to Git's binary index file format, including checksum validation.
- Initialization sets up the minimal `.git` directory required for other operations to function.
- Objects are stored compressed under `.git/objects` following the standard Git object storage layout.
  
---

## ASCII Diagram of Commit Object Creation Process

```
[Working Directory] ----
                        |
                     (add files)
                        |
                   [Index Entries]
                        |
                   (write_tree())
                        |
                 [Tree Object SHA-1]
                        |
                (get_local_master_hash())
                        |
                  [Parent Commit SHA-1]
                        |
                     (commit())
                        |
                 [New Commit Object]
                        |
          Update refs/heads/master to point here
```

---

This completes the technical reference documentation for repository initialization and status reporting as implemented in `repo.md`. The described functions provide essential capabilities enabling repository setup, file tracking, and commit management within the pygit system.