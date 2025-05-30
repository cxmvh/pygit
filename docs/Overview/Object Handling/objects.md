# objects.md

# Object Storage and Retrieval Functions Documentation

---

## Overview

This document provides comprehensive technical reference documentation for the core object storage and retrieval functions used in the `pygit` repository, specifically focusing on functionalities such as reading objects, hashing data to objects, finding objects, and related utilities. These functions are foundational for managing Git objects—blobs, trees, commits—and support multiple essential workflows including listing files, status reporting, diffs, repository initialization, and committing changes.

This file fits within the broader **Object Handling** section of the documentation tree, which covers Git object management at a detailed level. It is closely related to other files such as `cat_file.md`, which focuses on displaying Git objects, and `object_reading.md`, which details functions for reading and locating objects. The functions documented here are invoked across many core command flows like `pygit.ls_files`, `pygit.status`, `pygit.diff`, `pygit.init`, and `pygit.commit`.

---

## Function Documentation

### `read_object(sha1_prefix)`

#### Purpose
Reads a Git object from the object store based on its SHA-1 hash prefix and returns its type and data.

#### Parameters
- `sha1_prefix` (str): A prefix of the SHA-1 hash of the object to read. Must be at least 2 characters.

#### Behavior
- Locates the full object file path using `find_object`.
- Reads and decompresses the object data.
- Parses the object header to extract the object type (e.g., blob, tree, commit) and size.
- Validates that the size matches the decompressed data length.
- Returns a tuple `(obj_type, data_bytes)`.

#### Example Usage
```python
obj_type, data = read_object('a1b2c3d4')
print(f"Object type: {obj_type}")
print(f"Data bytes: {data[:50]}...")  # Print first 50 bytes
```

#### Internal Flow Diagram
```
+-------------------+
| sha1_prefix input |
+---------+---------+
          |
          v
+-----------------------+
| find_object(sha1_prefix)|
+-----------+------------+
            |
            v
+--------------------+
| read_file(object_path)|
+-----------+--------+
            |
            v
+-----------------------+
| zlib.decompress(data) |
+-----------+-----------+
            |
            v
+--------------------------+
| Parse header & extract   |
| obj_type and data        |
+--------------------------+
            |
            v
+------------------------+
| Return (obj_type, data)|
+------------------------+
```

---

### `hash_object(data, obj_type, write=True)`

#### Purpose
Computes the SHA-1 hash of object data of a given type and optionally writes it into the Git object store.

#### Parameters
- `data` (bytes): Raw data to hash.
- `obj_type` (str): Type of the object (`'blob'`, `'tree'`, `'commit'`).
- `write` (bool, default=True): Whether to write the object data to the object store.

#### Behavior
- Constructs the Git object header as `<type> <length>\0`.
- Concatenates header and data to form the full object representation.
- Computes SHA-1 hash of the full object.
- If `write` is True, compresses and writes the object to `.git/objects/`.
- Returns the SHA-1 hash as a hex string.

#### Example Usage
```python
data = b"Hello, Git Object!"
sha1 = hash_object(data, 'blob')
print(f"Object hash: {sha1}")
```

---

### `find_object(sha1_prefix)`

#### Purpose
Finds the file path of a Git object in the object store given a SHA-1 prefix.

#### Parameters
- `sha1_prefix` (str): The prefix of the SHA-1 hash of the object to find. Must be at least 2 characters.

#### Behavior
- Checks the `.git/objects/<first_two_chars>/` directory for files starting with the remaining prefix.
- Raises `ValueError` if no matching objects or multiple matches are found.
- Returns the full path to the object file.

#### Example Usage
```python
path = find_object('a1b2c3')
print(f"Object path: {path}")
```

---

### `read_index()`

#### Purpose
Reads the Git index file and returns a list of `IndexEntry` objects representing the current index state.

#### Parameters
- None

#### Behavior
- Reads the `.git/index` file.
- Validates the checksum and the index file signature.
- Parses each index entry, including metadata and file path.
- Returns a list of `IndexEntry` instances.

#### Example Usage
```python
entries = read_index()
for entry in entries:
    print(f"{entry.mode:o} {entry.sha1.hex()} {entry.path}")
```

---

### `write_index(entries)`

#### Purpose
Writes a list of `IndexEntry` objects to the Git index file.

#### Parameters
- `entries` (list of IndexEntry): List of index entries to write.

#### Behavior
- Packs each entry into the binary format expected by Git.
- Writes a header with version and entry count.
- Calculates and appends SHA-1 checksum.
- Writes the data to `.git/index`.

#### Example Usage
```python
entries = read_index()
# modify entries as needed
write_index(entries)
```

---

### `read_tree(sha1=None, data=None)`

#### Purpose
Reads a tree object and returns a list of tuples representing its entries.

#### Parameters
- `sha1` (str, optional): SHA-1 of the tree object to read.
- `data` (bytes, optional): Raw data of a tree object.

#### Behavior
- If `sha1` is provided, reads the object data and asserts it is a tree.
- Parses entries consisting of mode, path, and SHA-1.
- Returns a list of `(mode, path, sha1)` tuples.

#### Example Usage
```python
entries = read_tree(sha1='f3a1...')
for mode, path, sha1 in entries:
    print(f"{mode:o} {path} {sha1}")
```

---

### `write_tree()`

#### Purpose
Writes a tree object from the current index entries.

#### Parameters
- None

#### Behavior
- Reads all index entries.
- Assembles tree entries by concatenating mode, path, and SHA-1.
- Hashes and writes the tree object.
- Returns the SHA-1 hash of the tree.

#### Example Usage
```python
tree_sha1 = write_tree()
print(f"Tree object created with SHA-1: {tree_sha1}")
```

---

### `find_tree_objects(tree_sha1)`

#### Purpose
Recursively finds all object hashes contained in a tree (including the tree itself).

#### Parameters
- `tree_sha1` (str): SHA-1 hash of the tree object.

#### Behavior
- Reads the tree entries.
- For each entry that is a directory, recursively collects contained objects.
- Returns a set of SHA-1 hashes.

#### Example Usage
```python
objects = find_tree_objects(tree_sha1)
print(f"Objects in tree: {objects}")
```

---

### `find_commit_objects(commit_sha1)`

#### Purpose
Recursively finds all object hashes reachable from a commit (its tree, parents, and itself).

#### Parameters
- `commit_sha1` (str): SHA-1 hash of the commit object.

#### Behavior
- Reads the commit object, extracts tree and parent commits.
- Recursively collects all objects from the tree and parents.
- Returns a set of SHA-1 hashes.

#### Example Usage
```python
commit_objects = find_commit_objects(commit_sha1)
print(f"Objects in commit: {commit_objects}")
```

---

### `commit(message, author=None)`

#### Purpose
Creates a new commit from the current index with the given message.

#### Parameters
- `message` (str): The commit message.
- `author` (str, optional): Author string in the format `"Name <email>"`. If not provided, environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL` are used.

#### Behavior
- Writes the current tree object.
- Retrieves the current master branch commit to set as parent.
- Constructs the commit object content including tree, parent(s), author, committer, and message.
- Hashes and writes the commit object.
- Updates the `refs/heads/master` reference.
- Prints the commit SHA-1 and returns it.

#### Example Usage
```python
sha1 = commit("Initial commit")
print(f"Created commit {sha1}")
```

---

### `ls_files(details=False)`

#### Purpose
Lists files in the Git index.

#### Parameters
- `details` (bool, default=False): If True, print detailed information including mode, SHA-1, and stage number.

#### Behavior
- Reads index entries.
- Prints either just the file paths or detailed info per file.

#### Example Usage
```python
ls_files(details=True)
```

Output example:
```
100644 a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0 0  README.md
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

#### Purpose
Determines which objects from a local commit are missing on a remote repository.

#### Parameters
- `local_sha1` (str): SHA-1 hash of the local commit.
- `remote_sha1` (str or None): SHA-1 hash of the remote commit, or None if remote has no commits.

#### Behavior
- Gets sets of objects reachable from local and remote commits.
- Returns the difference (objects in local but not in remote).

#### Example Usage
```python
missing = find_missing_objects(local_sha1='abc123...', remote_sha1=None)
print(f"Missing objects to push: {missing}")
```

---

### `encode_pack_object(obj)`

#### Purpose
Encodes a single Git object for inclusion in a pack file.

#### Parameters
- `obj` (str): SHA-1 hash of the object to encode.

#### Behavior
- Reads the object type and data.
- Constructs variable-length header encoding type and size.
- Compresses the data.
- Returns concatenated header and compressed data bytes.

#### Example Usage
```python
pack_bytes = encode_pack_object('abc123...')
```

---

### `push(git_url, username=None, password=None)`

#### Purpose
Pushes the local master branch to a remote Git repository.

#### Parameters
- `git_url` (str): URL of the remote Git repository.
- `username` (str, optional): Username for authentication (defaults from environment).
- `password` (str, optional): Password for authentication (defaults from environment).

#### Behavior
- Gets remote and local master commit hashes.
- Finds objects missing on remote.
- Creates packfile data with missing objects.
- Sends push request over HTTP.
- Validates server response.
- Returns remote SHA-1 and set of missing objects pushed.

#### Example Usage
```python
push('https://example.com/myrepo.git')
```

---

### `read_file(path)`

#### Purpose
Reads bytes content from a file.

#### Parameters
- `path` (str): File path to read.

#### Behavior
- Opens the file in binary mode and returns all bytes.

#### Example Usage
```python
data = read_file('.git/HEAD')
print(data.decode())
```

---

### `write_file(path, data)`

#### Purpose
Writes bytes data to a file.

#### Parameters
- `path` (str): Path to the file.
- `data` (bytes): Data to write.

#### Behavior
- Opens the file in binary write mode and writes data.

#### Example Usage
```python
write_file('.git/HEAD', b'ref: refs/heads/master\n')
```

---

## Additional Notes

- Many functions depend on low-level file operations and zlib compression/decompression for object storage.
- Index entries use a packed binary structure conforming to Git's index file format.
- Object paths are organized in `.git/objects/` by splitting SHA-1 into directory and filename (first two chars as directory).
- `commit` and `write_tree` functions bridge index state to Git objects, enabling repository history management.
- Recursive functions like `find_tree_objects` and `find_commit_objects` allow traversal of object graphs.

---

## ASCII Diagram: Object Storage Layout

```
.git/
  objects/
    aa/
      bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
    bb/
      cccccccccccccccccccccccccccccccccccccccc
  refs/
    heads/
      master
  index
  HEAD
```

- Object files are stored under `.git/objects/<first_two_chars>/<remaining_38_chars>`.
- `index` holds the staging area entries.
- `refs/heads/master` points to the latest commit hash on the master branch.

---

This concludes the technical reference documentation for the core object handling functions in pygit, enabling developers and maintainers to understand, extend, and maintain the Git object storage and retrieval features.