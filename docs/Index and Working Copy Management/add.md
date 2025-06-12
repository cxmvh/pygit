# add.md

# Documentation for the `add` Function to Add Files to the Index

---

## Overview

The `add.md` file documents the `add` function, a core utility within the **Index and Working Copy Management** section of the repository. This function is responsible for adding specified files to the Git index (staging area), preparing them for future commits. It plays a crucial role in managing changes to the working directory by updating the index file to reflect the current state of files. The ability to add files accurately to the index ensures that subsequent commit operations capture intended changes effectively.

Within the broader documentation tree, `add.md` complements related files such as `index.md`, `index_management.md`, and `write_index.md` by focusing specifically on the staging operation. It integrates with other core functionalities such as hashing objects, reading/writing the index, and computing diffs, thereby forming a foundational aspect of Git repository manipulation in the `pygit` implementation.

---

## Function Documentation

### `add(paths)`

#### Purpose

The `add` function stages one or more files by adding them to the Git index. This involves hashing the file contents as Git blob objects, collecting file metadata, and updating the index entries accordingly. Staging files via `add` is a prerequisite step before committing changes to the repository.

#### Parameters

- **paths** (`list` of `str`): A list of file paths to be added to the Git index. File paths may use either forward slashes (`/`) or backward slashes (`\`) as separators; the function normalizes these paths internally.

#### Preconditions

- The specified files must exist in the working directory.
- The `.git/index` file should be readable and writable, or if missing, an empty index will be created.
- The environment must allow file reading and writing operations.

#### Operation Details

1. **Path Normalization:**  
   The function begins by normalizing all input file paths to use forward slashes (`/`), ensuring consistency across platforms.

2. **Read Current Index:**  
   It reads the existing Git index entries using `read_index()`. If the index file does not exist, it treats the index as empty.

3. **Filter Existing Entries:**  
   Entries corresponding to the files specified in `paths` are removed from the current index entries, effectively replacing any prior versions of these files.

4. **Process Each File:**
   - Reads the file content as bytes.
   - Hashes the content as a `blob` object using `hash_object()`, which stores the object in the Git object store.
   - Retrieves file metadata such as:
     - Creation time (`ctime`)
     - Modification time (`mtime`)
     - Device ID, inode number, mode, user ID, group ID, file size
   - Computes the flags field based on the path length (limited to 12 bits).
   - Constructs a new `IndexEntry` instance encapsulating all gathered data.

5. **Update and Sort Index Entries:**  
   The new entries are appended to the filtered list of existing entries, then sorted lexicographically by the file path to maintain order.

6. **Write Updated Index:**  
   The updated list of index entries is written back to the `.git/index` file using `write_index()`.

#### Example Usage

```python
# Example: Adding two files to the Git index
files_to_add = ['src/main.py', 'README.md']
add(files_to_add)
print("Files added to index successfully.")
```

---

## ASCII Diagram: Index Entry Update Flow

```
+----------------+       +----------------+       +----------------+
| Input File(s)  | --->  | Normalize Paths| --->  | Read Existing  |
|  (paths list)  |       |  (replace '\'  |       |  Index Entries |
|                |       |   with '/')    |       | (read_index()) |
+----------------+       +----------------+       +----------------+
                                                   |
                                                   v
                                          +----------------+
                                          | Remove Entries |
                                          |  for Input     |
                                          |  Paths         |
                                          +----------------+
                                                   |
                                                   v
                                          +----------------+
                                          | For Each Path: |
                                          | - Read Content |
                                          | - Hash Blob    |
                                          | - Get Metadata |
                                          | - Create Entry |
                                          +----------------+
                                                   |
                                                   v
                                          +----------------+
                                          | Append Entries |
                                          | Sort by Path   |
                                          +----------------+
                                                   |
                                                   v
                                          +----------------+
                                          | Write Index    |
                                          | (write_index)  |
                                          +----------------+
```

---

## Related Functions and Integration

- **`read_index()`**: Reads the current Git index entries. Used to retrieve existing staged files before updating.  
- **`hash_object(data, 'blob')`**: Hashes file content as a blob object and stores it. Returns the SHA-1 hash.  
- **`write_index(entries)`**: Writes a list of `IndexEntry` objects back to the `.git/index` file, updating the index on disk.  
- **`read_file(path)`**: Reads raw bytes of a file, used internally to get file content.  
- **`IndexEntry`**: Data structure representing a single entry in the Git index, including metadata and path.

---

## Summary

The `add` function is essential for staging file changes within the Git workflow. It bridges the working directory and the Git index by capturing file contents and metadata and updating the index accordingly. This documentation provides a comprehensive explanation of `add`'s purpose, parameters, detailed operation, and example usage, enabling developers and users to understand and utilize this functionality effectively within the `pygit` project.