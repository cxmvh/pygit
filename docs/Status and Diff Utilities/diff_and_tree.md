# diff_and_tree.md

## Overview

This document covers the core functionalities related to displaying diffs and managing tree objects within the pygit project. It provides reference information and usage details for key functions such as `diff`, `read_tree`, `write_tree`, and `ls_files`. These functions play a crucial role in showing changes between commits or the working directory and handling Git tree objects that represent directory snapshots. This file fits into the broader **Status and Diff Utilities** section of the documentation tree, complementing other documents that focus on repository status, file listing, and commit management. Understanding these functions is essential for developers working on repository inspection, status reporting, and object management features.

---

## Functions

### `diff`

**Purpose:**  
The `diff` function generates a difference view between two Git trees or between a tree and the working directory. It identifies changes such as added, deleted, modified, or renamed files, providing detailed information about the differences in file contents and metadata.

**Parameters:**  
- `old_tree` (optional): The tree object representing the old state (e.g., a commit or tree SHA). If omitted, the working directory or index may be used as the comparison baseline.  
- `new_tree` (optional): The tree object representing the new state to compare against.  
- `path` (optional): A specific path or subdirectory within the trees to limit the diff scope.

**Operation:**  
1. Reads the tree objects for both the old and new states using `read_tree`.  
2. Compares entries (files/directories) in both trees recursively.  
3. Detects added, deleted, and modified files by comparing file metadata and contents.  
4. For modified files, computes line-by-line differences to display changes.  
5. Optionally limits output to a specified path within the trees.

**Example Usage:**

```python
# Show diff between HEAD tree and working directory
diff(old_tree='HEAD', new_tree=None)

# Show diff between two specific commits by their tree SHAs
diff(old_tree='a1b2c3d4', new_tree='e5f6g7h8')

# Show diff limited to a subdirectory 'src/'
diff(old_tree='HEAD', new_tree=None, path='src/')
```

**ASCII Diagram:**

```
Old Tree (commit A)                      New Tree (commit B)
+-----------------+                     +-----------------+
| file1.txt       |                     | file1.txt       |  (modified)
| file2.txt       |                     | file2.txt       |  (deleted)
| src/            |                     | src/            |
|  └─ main.py     |                     |  └─ main.py     |  (modified)
+-----------------+                     |  └─ util.py     |  (added)
                                       +-----------------+

diff() identifies:
- file2.txt deleted
- file1.txt modified
- src/main.py modified
- src/util.py added
```

---

### `read_tree`

**Purpose:**  
`read_tree` reads a Git tree object from the repository and returns its contents in a structured format. This function is fundamental for accessing the directory snapshot stored in a tree object, enabling further operations like diffing, writing, or listing files.

**Parameters:**  
- `tree_sha`: The SHA-1 hash of the tree object to be read.

**Operation:**  
1. Retrieves the raw tree object data from the object database using the SHA.  
2. Parses the tree object entries, extracting mode, type (blob or tree), SHA, and file/directory name.  
3. Returns a data structure (e.g., dictionary or list) representing the tree contents, including nested trees.

**Example Usage:**

```python
tree_contents = read_tree('a1b2c3d4e5f6...')
for entry in tree_contents:
    print(f"{entry['mode']} {entry['type']} {entry['sha']} {entry['name']}")
```

---

### `write_tree`

**Purpose:**  
`write_tree` creates a new Git tree object from a given set of files and directories (usually from the index or working directory) and writes it to the object database. This function is essential for capturing the state of the working directory or index as a Git tree object.

**Parameters:**  
- `files`: A data structure representing files and directories to include in the tree, typically including file modes, content SHAs, and filenames.

**Operation:**  
1. Recursively processes the input files and directories to build tree entries.  
2. Encodes each entry with its mode, name, and SHA.  
3. Concatenates entries to form the raw tree object content.  
4. Writes the tree object to the database and returns its SHA.

**Example Usage:**

```python
files = [
    {'mode': '100644', 'sha': 'deadbeef...', 'name': 'README.md'},
    {'mode': '040000', 'sha': 'cafebabe...', 'name': 'src'},
]

tree_sha = write_tree(files)
print(f"Created tree object with SHA: {tree_sha}")
```

---

### `ls_files`

**Purpose:**  
The `ls_files` function lists files in the index or working directory, optionally filtering by path or status. It is used to show tracked files, their staged status, and other metadata, helping users understand current repository contents.

**Parameters:**  
- `path` (optional): Filters the list to files under a specific path or directory.  
- `staged` (optional): If True, lists only staged files.

**Operation:**  
1. Reads the index file to obtain tracked files and their metadata.  
2. Applies filtering based on parameters.  
3. Returns or prints a list of files with their statuses and hashes.

**Example Usage:**

```python
# List all files tracked in the index
ls_files()

# List files staged for the next commit under the 'docs/' directory
ls_files(path='docs/', staged=True)
```

---

## Summary Diagram of Tree Object Management and Diffing Flow

```
+-------------------+
|   Working Dir /    |
|   Index Files      |
+---------+---------+
          |
          | write_tree(files) -> tree object SHA
          v
+-------------------+       read_tree(tree_sha)       +-----------------+
| Tree Object Store  | <------------------------------ | Tree Object     |
| (object database)  |                                 | (parsed entries)|
+-------------------+                                 +-----------------+
          |                                                  |
          |                                                  |
          |                      diff(old_tree, new_tree)    |
          +--------------------------------------------------+
                                   |
                       +-------------------------------+
                       | Computes file changes & diffs |
                       +-------------------------------+
                                   |
                          Displays human-readable diff
```

---

*For additional details on related concepts such as object management and commit handling, please refer to the other documents in the [Status and Diff Utilities](./status_and_diff.md) and [Commit Management](./commit.md) sections.*