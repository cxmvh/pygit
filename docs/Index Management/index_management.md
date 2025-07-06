# Index Management

## Overview

This document provides detailed reference information for managing the Git index (also known as the staging area) within the repository. It covers functions and procedures for reading the index file, writing changes back to it, listing the files currently staged, and adding new or modified files to the index. These capabilities are essential for preparing changes to be committed, bridging the working directory and the commit history.

Index management is a critical intermediary step in Git's workflow, ensuring that changes are organized and recorded accurately before commit creation. This documentation fits into the broader "Index Management" section of the project, complementing related documentation on repository initialization, object storage, and basic Git operations.

---

## Functions

### `read_index(repo_path: str) -> List[IndexEntry]`

Reads the Git index file from the repository at `repo_path` and parses its contents into a list of index entries.

#### Purpose

- To load the current state of the index (staging area) into memory.
- Enables inspection and manipulation of the staged files before committing.

#### Parameters

- `repo_path` — The filesystem path to the root of the Git repository (where the `.git` directory resides).

#### Operation

1. Locate the `.git/index` file within the specified repository path.
2. Open and read the binary contents of the index file.
3. Parse the header to verify the index version and entry counts.
4. Iterate through each index entry and extract file metadata such as:
   - File mode
   - Object hash (SHA-1)
   - File path relative to the repository root
   - Timestamps and file stat information
5. Return a list of structured `IndexEntry` objects representing each staged file.

#### Example Usage

```python
index_entries = read_index('/path/to/repo')
for entry in index_entries:
    print(f"{entry.path} - {entry.sha.hex()} - mode: {oct(entry.mode)}")
```

---

### `write_index(repo_path: str, entries: List[IndexEntry]) -> None`

Writes the provided list of index entries back to the `.git/index` file, updating the staging area.

#### Purpose

- To persist changes made to the index in memory back to the repository's index file.
- Ensures that additions, removals, or modifications to the index are saved for subsequent Git operations.

#### Parameters

- `repo_path` — The filesystem path to the root of the Git repository.
- `entries` — A list of `IndexEntry` objects representing the desired state of the index.

#### Operation

1. Serialize the list of index entries into the Git index file format.
2. Compose the index header with the proper version and entry count.
3. Write each entry's metadata, SHA-1 hash, and path in the prescribed binary format.
4. Compute and append a SHA-1 checksum for index integrity verification.
5. Overwrite the existing `.git/index` file atomically to avoid corruption.

#### Example Usage

```python
# Assuming `entries` is a modified list of IndexEntry objects
write_index('/path/to/repo', entries)
```

---

### `ls_files(repo_path: str) -> List[str]`

Lists the file paths currently staged in the Git index.

#### Purpose

- To provide a simple interface for retrieving the list of all files that are currently in the staging area.
- Useful for commands like `git ls-files` which display staged file information.

#### Parameters

- `repo_path` — The filesystem path to the root of the Git repository.

#### Operation

1. Call `read_index` internally to load the index entries.
2. Extract the file path from each index entry.
3. Return a list of these file paths as strings.

#### Example Usage

```python
staged_files = ls_files('/path/to/repo')
print("Staged files:")
for filepath in staged_files:
    print(filepath)
```

---

### `add_file(repo_path: str, file_path: str) -> None`

Adds or updates a file in the Git index, staging it for the next commit.

#### Purpose

- To stage new or modified files by hashing their contents and updating the index accordingly.
- Reflects the equivalent of the `git add` command in this simplified Git implementation.

#### Parameters

- `repo_path` — The filesystem path to the root of the Git repository.
- `file_path` — The path to the file to add, relative to the repository root.

#### Operation

1. Read the file content from the working directory.
2. Compute the SHA-1 hash of the file contents.
3. Write the file contents as a blob object into the Git object database.
4. Load the current index entries using `read_index`.
5. If the file already exists in the index, replace its entry; otherwise, append a new entry.
6. Update the entry's metadata (timestamps, mode, SHA-1 hash).
7. Write the updated list of entries back using `write_index`.

#### Example Usage

```python
add_file('/path/to/repo', 'src/main.py')
print("File 'src/main.py' added to the index.")
```

---

## ASCII Diagram: Git Index in the Context of Git Workflow

```
+-----------------+        git add         +------------------+       git commit      +------------------+
| Working Directory|  ------------------>   |    Git Index     |  ----------------->  |   Git Repository  |
| (Unstaged files) |                        | (Staging Area)   |                      | (Committed Files) |
+-----------------+                        +------------------+                      +------------------+
        |                                           ^
        |                                           |
        |             git status / ls-files         |
        +-------------------------------------------+
```

This diagram illustrates the role of the Git index as the staging area where files are prepared after being added (`git add`) and before being committed (`git commit`).

---

## Related Documentation

- See the [repository_initialization.md](../Repository%20Initialization%20and%20Structure/repository_initialization.md) file for details on repository setup.
- Refer to [object_storage.md](../Object%20Storage%20and%20Management/object_storage.md) for information on Git object handling related to blobs created during indexing.
- For commit operations that use the index, see [commit.md](../Basic%20Git%20Operations/commit.md).

---

*End of index_management.md*