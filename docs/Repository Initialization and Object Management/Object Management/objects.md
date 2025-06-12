# objects.md - Technical Reference Documentation

## Overview

The `objects.md` file consolidates comprehensive documentation regarding Git object handling within the `pygit` project. It covers critical operations such as hashing, reading, writing, encoding, and locating Git objects, alongside the implementation details of the `cat_file` command. Functionally, this file is situated within the "Object Management" sub-section of "Repository Initialization and Object Management." The content serves as a vital resource for understanding the lifecycle of Git objects — from their creation and storage to retrieval and display — underpinning core Git functionality implemented in `pygit`.

---

## Function Documentation

### 1. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of given object data of a specified Git object type (e.g., `'blob'`, `'tree'`, `'commit'`). Optionally writes the compressed object data to the Git object store.

**Parameters:**  
- `data` (`bytes`): Raw object data bytes.  
- `obj_type` (`str`): Type of the object (e.g., `'blob'`, `'tree'`, `'commit'`).  
- `write` (`bool`, optional): If `True`, writes the compressed object data to the `.git/objects` directory. Defaults to `True`.

**Operation:**  
1. Construct a header string of the format `<obj_type> <len(data)>` and encode it as bytes.  
2. Concatenate the header, a null byte `b'\x00'`, and the data to form the full object data.  
3. Compute the SHA-1 hash of the full object data.  
4. If `write` is `True`, compress the full object data using zlib and write it to `.git/objects/xx/yyyy...` where `xx` is the first two hex chars of the hash and `yyyy...` is the remainder.  
5. Return the hex digest of the SHA-1 hash.

**Example Usage:**
```python
data = b'Hello, Git!'
obj_type = 'blob'
sha1_hash = hash_object(data, obj_type)
print(f'Object stored with SHA-1: {sha1_hash}')
```

---

### 2. `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object from the object store using a SHA-1 prefix and returns its type and data content.

**Parameters:**  
- `sha1_prefix` (`str`): Prefix of the SHA-1 hash identifying the object.

**Operation:**  
1. Locate the object file path by resolving the SHA-1 prefix in `.git/objects`.  
2. Read and decompress the object data using zlib.  
3. Parse the header to extract the object type and size.  
4. Extract the object data following the header.  
5. Validate the data length matches the size.  
6. Return a tuple `(obj_type, data_bytes)`.

**Example Usage:**
```python
obj_type, data = read_object('e69de29bb2')  # SHA-1 prefix for an empty blob
print(f'Object type: {obj_type}')
print(f'Data: {data}')
```

---

### 3. `write_file(path, data)`

**Purpose:**  
Writes raw bytes data to a file at the specified path.

**Parameters:**  
- `path` (`str`): File path to write data to.  
- `data` (`bytes`): Data bytes to write.

**Operation:**  
Open the file at `path` in binary write mode and write the `data` bytes.

**Example Usage:**
```python
write_file('.git/HEAD', b'ref: refs/heads/master')
```

---

### 4. `read_file(path)`

**Purpose:**  
Reads binary contents from a file at the specified path.

**Parameters:**  
- `path` (`str`): Path of the file to read.

**Operation:**  
Open the file in binary read mode and return its content as bytes.

**Example Usage:**
```python
content = read_file('.git/HEAD')
print(content.decode())
```

---

### 5. `find_object(sha1_prefix)`

**Purpose:**  
Finds the path to a Git object in the object store given a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (`str`): Prefix of the SHA-1 hash.

**Operation:**  
1. Verify prefix length is at least 2 characters.  
2. Look inside `.git/objects/<first_two_chars>/` directory for files starting with the remaining prefix.  
3. If no objects or multiple objects match, raise `ValueError`.  
4. Return the full file path of the matching object.

**Example Usage:**
```python
obj_path = find_object('e69de29bb2')
print(f'Object path: {obj_path}')
```

---

### 6. `read_tree(sha1=None, data=None)`

**Purpose:**  
Reads a Git tree object and parses it into a list of entries.

**Parameters:**  
- `sha1` (`str`, optional): SHA-1 hash of the tree object.  
- `data` (`bytes`, optional): Raw tree object data bytes.

**Operation:**  
- If `sha1` is provided, read the object data for the tree.  
- If `data` is provided, parse it directly.  
- Parse entries one by one: each entry contains a mode, path, and SHA-1 hash.  
- Return a list of tuples `(mode, path, sha1)`.

**Example Usage:**
```python
entries = read_tree(sha1='a1b2c3d4...')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

### 7. `cat_file(mode, sha1_prefix)`

**Purpose:**  
Shows content or information about a Git object identified by SHA-1 prefix, mimicking the `git cat-file` command.

**Parameters:**  
- `mode` (`str`): Mode of operation (`'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, or `'pretty'`).  
- `sha1_prefix` (`str`): SHA-1 prefix of the object.

**Operation:**  
- Read the object type and data.  
- Depending on `mode`:
  - If `'commit'`, `'tree'`, or `'blob'`: output raw object data.  
  - If `'size'`: print the size of the object data.  
  - If `'type'`: print the type of the object.  
  - If `'pretty'`: for `'commit'` and `'blob'`, output raw data; for `'tree'`, output a formatted list of entries.

**Example Usage:**
```python
cat_file('pretty', 'e69de29bb2')
```

---

### 8. `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects to the Git index file.

**Parameters:**  
- `entries` (list of `IndexEntry`): List of index entries to write.

**Operation:**  
1. For each entry, pack fields according to Git index format.  
2. Encode the path and pad the entry to an 8-byte boundary.  
3. Combine all entries with the index header.  
4. Append SHA-1 checksum of the entire index data.  
5. Write the combined data to `.git/index`.

**Example Usage:**
```python
write_index(entries)
```

---

### 9. `read_index()`

**Purpose:**  
Reads the Git index file and returns a list of `IndexEntry` objects.

**Parameters:**  
None.

**Operation:**  
1. Read the `.git/index` file.  
2. Validate the signature and version.  
3. Verify the SHA-1 checksum.  
4. Parse each index entry and decode the path.  
5. Return the list of entries.

**Example Usage:**
```python
entries = read_index()
for entry in entries:
    print(entry.path)
```

---

### 10. `add(paths)`

**Purpose:**  
Adds files to the Git index by hashing their contents and updating the index entries.

**Parameters:**  
- `paths` (list of str): File paths to add to the index.

**Operation:**  
1. Normalize paths to use forward slashes.  
2. Read current index entries, excluding those matching paths to add.  
3. For each path, hash the file as a blob object.  
4. Create a new `IndexEntry` for each file with metadata and SHA-1.  
5. Append and sort entries, then write back the index.

**Example Usage:**
```python
add(['README.md', 'setup.py'])
```

---

### 11. `write_tree()`

**Purpose:**  
Creates a tree object from the current index entries (only supports a flat directory) and writes it to the object store.

**Parameters:**  
None.

**Operation:**  
1. Read the index entries.  
2. For each entry, format the mode and path, concatenated with the SHA-1 hash.  
3. Concatenate all entries and hash them as a `'tree'` object.  
4. Return the SHA-1 of the tree object.

**Example Usage:**
```python
tree_sha1 = write_tree()
print(f'Tree object created: {tree_sha1}')
```

---

### 12. `commit(message, author=None)`

**Purpose:**  
Creates a commit object with the current state of the index and a commit message, then updates the master branch reference.

**Parameters:**  
- `message` (`str`): Commit message.  
- `author` (`str`, optional): Author information in `Name <email>` format.

**Operation:**  
1. Write the current index as a tree object.  
2. Get the current local master commit hash as parent.  
3. Compose commit metadata including author, committer, timestamp, and parent.  
4. Hash the commit object and write to the object store.  
5. Update `.git/refs/heads/master` with the new commit SHA-1.

**Example Usage:**
```python
commit('Initial commit')
```

---

### 13. `get_status()`

**Purpose:**  
Determines which files in the working directory have changed, are new, or deleted compared to the Git index.

**Parameters:**  
None.

**Operation:**  
1. Recursively scan working directory excluding `.git`.  
2. Read the index entries.  
3. Detect changed files by comparing blob hashes of working files to index.  
4. Identify new files (in working copy but not index).  
5. Identify deleted files (in index but not working copy).  
6. Return three sorted lists: `(changed_paths, new_paths, deleted_paths)`.

**Example Usage:**
```python
changed, new, deleted = get_status()
print('Changed:', changed)
print('New:', new)
print('Deleted:', deleted)
```

---

### 14. `status()`

**Purpose:**  
Prints a human-readable status report of the working copy files compared to the index.

**Parameters:**  
None.

**Operation:**  
1. Get changed, new, and deleted files via `get_status()`.  
2. Print categorized lists to standard output.

**Example Usage:**
```python
status()
```

---

### 15. `diff()`

**Purpose:**  
Displays unified diffs between the index and working copy for files that have changed.

**Parameters:**  
None.

**Operation:**  
1. Retrieve changed files via `get_status()`.  
2. For each changed file, read blob data from index and working file.  
3. Use Python's `difflib.unified_diff` to generate diff lines.  
4. Print diffs separated by dashed lines.

**Example Usage:**
```python
diff()
```

---

### 16. `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively collects SHA-1 hashes of all objects (trees and blobs) contained within a given tree object.

**Parameters:**  
- `tree_sha1` (`str`): SHA-1 of the tree object.

**Operation:**  
1. Add the root tree SHA-1 to the set.  
2. For each entry in the tree, if directory, recurse; else add blob SHA-1.  
3. Return the complete set of object SHA-1 hashes.

**Example Usage:**
```python
objects = find_tree_objects('a1b2c3d4...')
print(f'Objects in tree: {objects}')
```

---

### 17. `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively collects SHA-1 hashes of all objects referenced by a commit, including its tree, parents, and itself.

**Parameters:**  
- `commit_sha1` (`str`): SHA-1 of the commit object.

**Operation:**  
1. Add the commit SHA-1 to the set.  
2. Parse the commit object to get tree SHA-1 and parent commit SHA-1s.  
3. Recursively collect objects from the tree and parents.  
4. Return the complete set.

**Example Usage:**
```python
commit_objects = find_commit_objects('f7e8d9c0...')
print(f'Objects in commit: {commit_objects}')
```

---

### 18. `encode_pack_object(obj)`

**Purpose:**  
Encodes a Git object for inclusion in a pack file, including a variable-length header and compressed data.

**Parameters:**  
- `obj` (`str`): SHA-1 hash of the object.

**Operation:**  
1. Read the object type and data.  
2. Convert object type to numeric code.  
3. Encode size in variable-length format with continuation bits.  
4. Compress the object data.  
5. Return concatenated header and compressed data bytes.

**Example Usage:**
```python
packed_data = encode_pack_object('e69de29bb2')
# Use packed_data in pack files
```

---

### 19. `create_pack(objects)`

**Purpose:**  
Creates a Git pack file containing the specified set of object SHA-1 hashes.

**Parameters:**  
- `objects` (set of `str`): SHA-1 hashes of objects to include.

**Operation:**  
1. Pack file header with signature, version, and object count.  
2. Encode each object using `encode_pack_object`.  
3. Concatenate all encoded objects to form body.  
4. Append SHA-1 checksum of the entire pack content.  
5. Return full pack file data bytes.

**Example Usage:**
```python
pack_data = create_pack({'e69de29bb2', 'a1b2c3d4e5'})
write_file('packfile.pack', pack_data)
```

---

### 20. `push(git_url, username=None, password=None)`

**Purpose:**  
Pushes the local master branch to a remote Git repository.

**Parameters:**  
- `git_url` (`str`): URL of the remote Git repository.  
- `username` (`str`, optional): Username for authentication.  
- `password` (`str`, optional): Password for authentication.

**Operation:**  
1. Retrieve remote master commit hash.  
2. Retrieve local master commit hash.  
3. Find missing objects on remote compared to local.  
4. Create a pack file for missing objects.  
5. Perform authenticated HTTP POST to remote `/git-receive-pack`.  
6. Verify server response for success.  
7. Return tuple `(remote_sha1, missing_objects)`.

**Example Usage:**
```python
push('https://github.com/user/repo.git', username='user', password='pass')
```

---

### ASCII Diagram: Git Object Storage Structure

```
.git/
  objects/
    xx/          # directory named by first two hex characters of SHA-1
      yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy  # file named by remaining 38 hex chars (compressed object)
```

This directory structure allows efficient storage and lookup of objects by SHA-1 hash.

---

# Summary

This documentation covers the core functions responsible for Git object management within `pygit`. It details how objects are hashed, stored, located, read, and displayed, as well as how the index and commit workflows interact with these objects. The included `cat_file` command implementation demonstrates practical usage for inspecting Git objects. Together, these components provide a foundational understanding of Git's object model and its implementation in `pygit`.