# Object Handling in Git

This document provides comprehensive reference documentation for handling Git objects within the `pygit` project. It covers the mechanisms for hashing, encoding, reading, writing, and processing Git objects such as blobs, trees, and commits. These functionalities form the core of the "Object Handling" section in the overall documentation tree, enabling the manipulation and traversal of Git's internal data structures.

Understanding these functions is essential for developers working on repository data management, object storage, and Git operations such as commit creation, tree building, and repository synchronization.

---

## hash_object(data, obj_type, write=True)

Compute the SHA-1 hash for the given object data of a specified type and optionally write the compressed object to the Git object store.

### Parameters:
- `data` (bytes): The raw content of the object.
- `obj_type` (str): The type of the object (`'blob'`, `'tree'`, `'commit'`, etc.).
- `write` (bool, default `True`): If `True`, store the object in the `.git/objects` directory.

### Operation:
1. Create a header string as `"<obj_type> <len(data)>"` and encode it to bytes.
2. Concatenate the header, a null byte (`\x00`), and the data.
3. Compute the SHA-1 hash of the full data.
4. If `write` is `True`, store the compressed full data in the object store under `.git/objects/xx/yyyy...` where `xx` are the first two hex digits and `yyyy...` is the rest.
5. Return the SHA-1 hash as a 40-character hex string.

### Example:
```python
data = b'Hello, Git!'
obj_type = 'blob'
sha1_hash = hash_object(data, obj_type)
print(f'Object hash: {sha1_hash}')
```

---

## read_object(sha1_prefix)

Read a Git object by its SHA-1 prefix and return its type and raw data.

### Parameters:
- `sha1_prefix` (str): The prefix (at least 2 characters) of the object's SHA-1 hash.

### Operation:
1. Locate the object file in `.git/objects` using the prefix.
2. Decompress the file content.
3. Parse the header to extract the object type and size.
4. Extract the data bytes following the null byte.
5. Validate that the declared size matches the actual data length.
6. Return a tuple `(obj_type, data)`.

### Example:
```python
obj_type, data = read_object('a1b2c3d')
print(f'Type: {obj_type}')
print(data.decode())
```

---

## find_object(sha1_prefix)

Locate the path of an object file by its SHA-1 prefix.

### Parameters:
- `sha1_prefix` (str): At least 2-character prefix of the SHA-1 hash.

### Operation:
- Search `.git/objects/<first two chars>/` for matching files starting with the remaining prefix.
- Raise an error if no or multiple matches found.
- Return the full file path of the object.

---

## write_file(path, data)

Write binary data to a file at the specified path.

### Parameters:
- `path` (str): File path.
- `data` (bytes): Data to write.

---

## read_file(path)

Read the contents of a file as bytes.

### Parameters:
- `path` (str): File path.

### Returns:
- `bytes`: File content.

---

## read_index()

Read and parse the Git index file.

### Returns:
- List of `IndexEntry` objects representing staged files and metadata.

### Operation:
- Read `.git/index`.
- Validate checksum and header signature.
- Parse each entry's metadata and path.
- Return the list of entries.

---

## write_index(entries)

Write a list of `IndexEntry` objects back to the Git index file.

### Parameters:
- `entries` (list): List of `IndexEntry` objects.

### Operation:
- Pack each entry's binary representation.
- Write the packed data and SHA-1 checksum to `.git/index`.

---

## ls_files(details=False)

List files in the Git index.

### Parameters:
- `details` (bool): When `True`, print mode, SHA-1, stage, and path.

### Example:
```python
ls_files(details=True)
```

---

## add(paths)

Add given file paths to the Git index.

### Parameters:
- `paths` (list of str): File paths to add.

### Operation:
- Normalize paths.
- Read existing index entries excluding the new paths.
- For each new path:
  - Read file content.
  - Hash as blob object.
  - Create new `IndexEntry` with file metadata.
- Sort entries and write updated index.

---

## write_tree()

Create and write a tree object from current index entries.

### Returns:
- SHA-1 hash of the new tree object.

### Operation:
- For each top-level index entry:
  - Format mode and path.
  - Append SHA-1.
- Concatenate all entries.
- Hash and write as a `tree` object.
- Return tree hash.

---

## read_tree(sha1=None, data=None)

Parse a tree object from a SHA-1 or raw data.

### Parameters:
- `sha1` (str): SHA-1 hash of the tree object.
- `data` (bytes): Raw tree object data.

### Returns:
- List of tuples `(mode, path, sha1)` representing tree entries.

---

## commit(message, author=None)

Create a commit object from the current index state and write it.

### Parameters:
- `message` (str): Commit message.
- `author` (str, optional): Author string; defaults to environment variables.

### Returns:
- SHA-1 hash of the commit object.

### Operation:
- Write the tree object.
- Get parent commit hash.
- Construct commit metadata with author, timestamp, and message.
- Hash and write commit object.
- Update `refs/heads/master`.
- Print commit hash.

---

## get_local_master_hash()

Retrieve the current commit hash of the local master branch.

### Returns:
- SHA-1 string of the commit or `None` if no commit exists.

---

## get_status()

Determine the working copy status relative to the index.

### Returns:
- Tuple `(changed, new, deleted)` with lists of paths.

### Operation:
- Walk working directory excluding `.git`.
- Compare files' blob hashes with index.
- Identify changed, new, and deleted files.

---

## status()

Print the working copy status.

---

## diff()

Show diffs for files changed between the index and working copy.

### Operation:
- Get changed files via `get_status`.
- For each changed file:
  - Read blob from index and working copy.
  - Generate unified diff.
  - Print diff.

---

## cat_file(mode, sha1_prefix)

Print contents or information about a Git object.

### Parameters:
- `mode` (str): One of `'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, `'pretty'`.
- `sha1_prefix` (str): SHA-1 prefix of the object.

### Operation:
- Read object data.
- Depending on mode:
  - `'commit'`, `'tree'`, `'blob'`: print raw data.
  - `'size'`: print data size.
  - `'type'`: print object type.
  - `'pretty'`: print formatted content (e.g., tree entries).

---

## find_tree_objects(tree_sha1)

Recursively find all objects referenced by a tree.

### Parameters:
- `tree_sha1` (str): SHA-1 hash of a tree object.

### Returns:
- Set of SHA-1 hashes including the tree and all contained objects.

---

## find_commit_objects(commit_sha1)

Recursively find all objects referenced by a commit and its ancestors.

### Parameters:
- `commit_sha1` (str): SHA-1 hash of a commit object.

### Returns:
- Set of SHA-1 hashes including the commit, its tree, parent commits, and all referenced objects.

---

## find_missing_objects(local_sha1, remote_sha1)

Determine objects present locally but missing remotely.

### Parameters:
- `local_sha1` (str): Local commit SHA-1.
- `remote_sha1` (str or None): Remote commit SHA-1, or None if no remote commits.

### Returns:
- Set of SHA-1 hashes missing on remote.

---

## encode_pack_object(obj)

Encode a single object for inclusion in a Git pack file.

### Parameters:
- `obj` (str): SHA-1 hash of the object.

### Returns:
- Bytes representing the variable-length header and compressed object data.

---

## create_pack(objects)

Create a Git pack file containing the given objects.

### Parameters:
- `objects` (set): Set of SHA-1 hashes.

### Returns:
- Bytes of the complete pack file.

---

## push(git_url, username=None, password=None)

Push the local master branch to a remote repository.

### Parameters:
- `git_url` (str): URL of the remote Git repository.
- `username` (str, optional): Authentication username.
- `password` (str, optional): Authentication password.

### Operation:
- Retrieve remote master commit hash.
- Obtain local master commit hash.
- Determine missing objects on remote.
- Build pack file.
- Send push request via HTTP.
- Validate response.

---

# ASCII Diagram: Object Storage Structure

```
.git/
├── objects/
│   ├── xx/
│   │   ├── yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy (compressed object file)
│   │   └── ...
│   └── ...
├── refs/
│   └── heads/
│       └── master (contains commit SHA-1)
├── index (staging area metadata)
└── HEAD (reference to current branch)
```

---

# Summary

This document outlines the core functions managing Git objects in the pygit implementation. These include:

- Hashing and storing objects (`hash_object`, `write_file`).
- Reading and parsing objects (`read_object`, `read_tree`, `find_object`).
- Index file manipulation (`read_index`, `write_index`, `add`).
- Commit and tree creation (`write_tree`, `commit`).
- Repository status and diffs (`get_status`, `status`, `diff`).
- Object traversal for commits and trees (`find_commit_objects`, `find_tree_objects`).
- Packfile creation and pushing (`encode_pack_object`, `create_pack`, `push`).

Together, these functions enable a robust Git object handling layer, facilitating repository management and synchronization.