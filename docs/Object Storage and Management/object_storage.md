# Object Storage and Management

This document provides a comprehensive reference on Git object handling within the repository. It covers the processes of hashing, storing, retrieving, and packaging Git objects such as blobs, trees, and commits. As part of the broader *Object Storage and Management* section, this file is essential for understanding how Git internally manages data integrity and structure, enabling efficient version control. The information here underpins higher-level commands like commit creation, file listing, and pushing changes.

---

## Git Object Hashing and Storage

Git stores content as objects in a content-addressable manner. Each object is identified by its SHA-1 hash computed from the object’s type, size, and content. This ensures object immutability and enables efficient storage and retrieval.

### `hash_object(data: bytes, obj_type: str, write: bool = False) -> str`

**Purpose:**  
Computes the SHA-1 hash of a Git object constructed from the provided data and type. Optionally writes the object to the repository’s object store.

**Parameters:**  
- `data`: The raw bytes content of the object (e.g., file contents for blobs).  
- `obj_type`: The type of the object (`'blob'`, `'tree'`, `'commit'`).  
- `write`: If `True`, the object is stored in the `.git/objects` directory.

**Operation Steps:**  
1. Construct the object header as `"{obj_type} {len(data)}\0"`.  
2. Concatenate the header and data to form the full object byte sequence.  
3. Compute the SHA-1 hash of this sequence.  
4. If `write` is `True`, compress the object using zlib and write it into the object store directory, split by the first two characters of the hash as a folder name, and the remaining 38 characters as the file name.

**Example Usage:**
```python
file_contents = b"Hello, Git!"
blob_sha = hash_object(file_contents, "blob", write=True)
print(f"Stored blob object with SHA-1: {blob_sha}")
```

---

## Object Retrieval

Retrieving objects involves locating the compressed file by its SHA-1 hash, decompressing, and parsing the object header and content.

### `read_object(sha: str) -> Tuple[str, bytes]`

**Purpose:**  
Reads and decompresses a Git object from the object store by its SHA-1 hash.

**Parameters:**  
- `sha`: The SHA-1 hash string of the desired object.

**Operation Steps:**  
1. Determine the object file path by splitting `sha` into directory (first two characters) and filename (remaining characters).  
2. Read and decompress the object file using zlib.  
3. Parse the header (format: `"<obj_type> <size>\0"`) to extract the object type and size.  
4. Return a tuple of the object type and raw data bytes.

**Example Usage:**
```python
obj_type, obj_data = read_object("e69de29bb2d1d6434b8b29ae775ad8c2e48c5391")
print(f"Object type: {obj_type}")
print(f"Object data length: {len(obj_data)} bytes")
```

---

## Tree Object Creation

Tree objects represent directory structures by storing entries for files and subdirectories, each referencing blob or tree objects.

### `write_tree(directory: str = '.') -> str`

**Purpose:**  
Creates a tree object representing the current state of files and directories under `directory` and writes it to the object store.

**Parameters:**  
- `directory`: Path to the directory to be represented as a tree (defaults to current directory).

**Operation Steps:**  
1. Recursively read the contents of `directory`.  
2. For each file, create a blob object and retrieve its SHA-1.  
3. For each subdirectory, recursively create a tree object and retrieve its SHA-1.  
4. Format entries as `"<mode> <filename>\0<sha_bytes>"` concatenated together.  
5. Hash and store the concatenated entries as a tree object.  
6. Return the SHA-1 hash of the tree object.

**Example Usage:**
```python
tree_sha = write_tree('.')
print(f"Created tree object: {tree_sha}")
```

---

## Commit Object Creation

Commit objects record a snapshot of the repository at a point in time, referencing a tree object along with metadata such as author and message.

### `commit_tree(tree_sha: str, message: str, parent: Optional[str] = None) -> str`

**Purpose:**  
Creates a commit object referencing a tree, optionally linking to a parent commit, and stores it.

**Parameters:**  
- `tree_sha`: SHA-1 hash of the tree object representing the commit snapshot.  
- `message`: Commit message string.  
- `parent`: Optional SHA-1 hash of the parent commit.

**Operation Steps:**  
1. Construct commit content including:  
   - `tree <tree_sha>`  
   - `parent <parent>` (if present)  
   - `author <author info> <timestamp> <timezone>`  
   - `committer <committer info> <timestamp> <timezone>`  
   - Blank line followed by the commit message.  
2. Hash and write the commit object.  
3. Return the commit SHA-1 hash.

**Example Usage:**
```python
commit_sha = commit_tree(tree_sha, "Initial commit")
print(f"Created commit object: {commit_sha}")
```

---

## Object Packaging for Push

Before pushing commits to a remote repository, Git packages objects into a packfile for efficient transfer.

### `pack_objects(sha_list: List[str]) -> bytes`

**Purpose:**  
Packs a list of objects by their SHA-1 hashes into a single compressed packfile byte stream.

**Parameters:**  
- `sha_list`: List of SHA-1 hashes of objects to include in the packfile.

**Operation Steps:**  
1. Collect and order objects based on dependencies (e.g., commits reference trees, trees reference blobs).  
2. Compress and delta-encode objects to minimize packfile size.  
3. Build packfile header and index.  
4. Return the bytes representing the complete packfile.

**Example Usage:**
```python
objects_to_pack = [commit_sha, tree_sha, blob_sha]
packfile_bytes = pack_objects(objects_to_pack)
# The packfile_bytes can then be sent to a remote repository
```

---

## ASCII Diagram: Git Object Storage Structure

```
.git/
 └── objects/
     ├── e6/
     │    └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391  # blob object file
     ├── 4b/
     │    └── 82d8b1f1f9a7a6b7e7a9b1e5f0f5a3c3d1c4f3  # tree object file
     └── ... 
```

- The first two characters of the SHA-1 form a directory name.  
- The remaining 38 characters form the file name containing the zlib-compressed object data.

---

This document serves as a foundational reference for developers working on or extending Git object handling, enabling deeper insights into the low-level mechanics of version control storage. For higher-level commands like `commit`, `push`, and `ls-files`, understanding these primitives is critical. For related commands that display or manipulate these objects, please refer to [`cat_file.md`](./cat_file.md) and [`git_operations.md`](./git_operations.md).