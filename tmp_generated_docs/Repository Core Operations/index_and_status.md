---
sidebar_position: 2
---

# Git Index Structure and Status Management

## Overview

This document explains the structure and management of the Git index, an essential component in Git's staging area that tracks file metadata and content snapshots before committing. It covers reading and writing the index file, adding files to the index, representing index entries, and listing files tracked in the index via `ls_files`. Additionally, it details how Git determines the working directory and index status by comparing file states to provide meaningful status information. The explanations integrate relevant code flows from `pygit.ls_files`, `pygit.status`, and `pygit.commit`. This file plays a critical role within the "Repository Core Operations" section, bridging repository initialization and commit creation by managing the staging area.

---

## Index File Representation and Management

The Git index is a binary file usually located at `.git/index`. It maintains a list of entries representing files staged for the next commit. Each index entry stores metadata such as file paths, timestamps, device and inode numbers, file modes, object hashes, and flags.

### Index Entry Structure

Each index entry contains the following fields:

- **ctime**: The file's last metadata change time (seconds and nanoseconds)
- **mtime**: The file's last modification time (seconds and nanoseconds)
- **dev**: Device number of the file system containing the file
- **ino**: Inode number of the file
- **mode**: File mode (e.g., regular file, executable, symlink)
- **uid**: User ID of the file owner
- **gid**: Group ID of the file owner
- **size**: File size in bytes
- **sha1**: SHA-1 hash of the file content stored as an object
- **flags**: Miscellaneous flags (including path length)
- **path**: Relative file path

Each entry is aligned to an 8-byte boundary.

### ASCII Diagram: Index File Layout

```
+---------------------------------------------------------+
| Header: signature, version, entry count                 |
+---------------------------------------------------------+
| Entry 1: ctime, mtime, dev, ino, mode, uid, gid, size,  |
|          sha1, flags, path (aligned to 8 bytes)         |
+---------------------------------------------------------+
| Entry 2: ...                                            |
+---------------------------------------------------------+
| ...                                                     |
+---------------------------------------------------------+
```

---

## Core Functions

### Reading the Index (`read_index`)

**Purpose**:  
Loads the index file from disk into memory, parsing its entries into structured objects for further manipulation.

**Parameters**:  
- `index_path` (str): Path to the index file, typically `.git/index`.

**Operation**:  
1. Open the index file in binary mode.  
2. Read and verify the header signature (`"DIRC"`), version number, and entry count.  
3. Iteratively read each entry, parsing fixed-length fields and variable-length path strings.  
4. Store parsed entries in a list or dictionary keyed by path for quick lookup.

**Example Usage**:

```python
index_entries = read_index('.git/index')
for entry in index_entries:
    print(f"File: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### Writing the Index (`write_index`)

**Purpose**:  
Serializes and writes the in-memory index entries back to the index file on disk, updating the staging area.

**Parameters**:  
- `index_path` (str): Path to save the index file.  
- `entries` (list): List of index entry objects to write.

**Operation**:  
1. Serialize the header with the signature, version, and number of entries.  
2. For each entry, serialize all fields including the path, ensuring 8-byte alignment.  
3. Calculate and append a SHA-1 checksum of the contents for integrity verification.  
4. Write the complete binary data to the file.

**Example Usage**:

```python
write_index('.git/index', index_entries)
```

---

### Adding Files to the Index (`add`)

**Purpose**:  
Stages files by hashing their content, creating or updating corresponding index entries.

**Parameters**:  
- `file_paths` (list): List of file paths to add to the index.  
- `index_entries` (list): Current list of index entries to update.

**Operation**:  
1. For each file path:  
   - Read the file content from the working directory.  
   - Hash the content to create a Git blob object.  
   - Create a new index entry or update the existing one with new metadata and hash.  
2. Update the in-memory index entries list.

**Example Usage**:

```python
add(['README.md', 'src/main.py'], index_entries)
write_index('.git/index', index_entries)
```

---

### Listing Files in the Index (`ls_files`)

**Purpose**:  
Returns a list of files currently staged in the index along with their metadata.

**Parameters**:  
- `index_entries` (list): The list of parsed index entries.

**Operation**:  
Iterate over index entries and extract file paths, SHA-1 hashes, and other relevant metadata.

**Example Usage**:

```python
for entry in ls_files(index_entries):
    print(f"{entry.sha1.hex()} {entry.path}")
```

---

### Determining Status (`status`)

**Purpose**:  
Computes the status of files by comparing the working directory files against the index entries and HEAD commit tree.

**Operation**:

1. Load the current index entries.  
2. For each file in the working directory:  
   - Check if it exists in the index.  
   - Compare file metadata (mtime, size) and contents hash with index entry.  
3. Determine status states such as:  
   - **Untracked**: File not in index.  
   - **Modified**: File differs from index snapshot.  
   - **Staged**: Changes present in index but not yet committed.  
   - **Unmodified**: File matches index snapshot.  
4. Optionally compare index with HEAD commit tree for staged/unstaged changes.

**Example Usage**:

```python
status_report = status(index_entries, working_directory_files, head_commit_tree)
for path, state in status_report.items():
    print(f"{path}: {state}")
```

---

## Integration with Commit Process

The index acts as the staging area for commits. When `pygit.commit` is invoked, it reads the index to create a tree object representing the staged snapshot. The commit records this tree along with metadata and parent commits. Managing the index correctly ensures that the commit includes exactly the intended changes.

---

## Summary ASCII Flow Diagram

```
Working Directory
       |
       v
   [add()]  -- hash files --> Create/Update Index Entries
       |
       v
  [write_index()] --> .git/index (binary file)
       |
       v
  [status()] compares working dir vs index vs HEAD for file states
       |
       v
  [commit()] reads index to create tree and commit objects
```

---

This documentation file serves as a comprehensive reference for developers working with the Git index and related status operations within the `pygit` implementation. For details on repository initialization, see [repository_and_initialization.md](repository_and_initialization.md), and for object handling and commit creation, consult [object_handling_and_storage.md](object_handling_and_storage.md) and [trees_and_commits.md](trees_and_commits.md).