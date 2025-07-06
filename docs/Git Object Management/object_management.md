```mdx
# Object Management in Git

This document provides an in-depth explanation of how Git objects are handled within the repository. It covers the hashing, writing, reading, and storage of various object types including blobs, trees, and commits. Additionally, it explains the encoding schemes used for pack files and details how objects are located and retrieved from the object database, including recursive retrieval through trees and commits.

Located within the **Git Object Management** section of the documentation, this file serves as a crucial reference for developers working on the core mechanisms of Git object storage and retrieval. Understanding these details is fundamental for contributing to Git internals or building tools that interact directly with Git repositories at the object level.

---

## Hashing and Storing Objects

### `hash_object(data: bytes, obj_type: str) -> str`

**Purpose:**  
Computes the SHA-1 hash of a Git object given its data and type, and prepares the object for storage in the object database.

**Parameters:**  
- `data`: The raw bytes of the object's content (e.g., file contents for a blob).  
- `obj_type`: The type of the object, one of `"blob"`, `"tree"`, or `"commit"`.

**Operation:**  
1. Prepend the object type and length to the data in the format:  
   ```
   <obj_type> <len(data)>\0<data>
   ```  
2. Compute the SHA-1 hash of this combined byte sequence.  
3. Store the compressed object under the `.git/objects/` directory using the hash as the key.

**Example Usage:**

```python
data = b"Hello, Git!"
obj_type = "blob"
sha1_hash = hash_object(data, obj_type)
print(f"Object stored with hash: {sha1_hash}")
```

**Notes:**  
- The SHA-1 hash serves as the unique identifier for the object.  
- Objects are stored compressed using zlib to save space.

---

## Writing Objects to the Object Database

### `write_object(data: bytes, obj_type: str) -> str`

**Purpose:**  
Writes a Git object to the object database and returns its SHA-1 hash.

**Parameters:**  
- `data`: The object's data as bytes.  
- `obj_type`: The type of the object (`"blob"`, `"tree"`, or `"commit"`).

**Step-by-step:**  
1. Use `hash_object` to compute the hash and prepare the object data.  
2. Check if the object already exists; if not, compress and write it to disk.  
3. Return the computed SHA-1 hash.

**Example Usage:**

```python
blob_data = b"Sample file content"
blob_hash = write_object(blob_data, "blob")
print(f"Blob object saved with hash: {blob_hash}")
```

---

## Reading Objects from the Database

### `read_object(sha1_hash: str) -> Tuple[str, bytes]`

**Purpose:**  
Retrieves and decompresses a Git object from the database given its SHA-1 hash.

**Parameters:**  
- `sha1_hash`: The SHA-1 hash string identifying the object.

**Returns:**  
- A tuple `(obj_type, data)`, where `obj_type` is the object type string and `data` is the raw bytes of the object content.

**How it works:**  
1. Locate the object file in `.git/objects/` using the hash prefix and suffix.  
2. Decompress the object data using zlib.  
3. Parse the header to extract the object type and size.  
4. Return the type and the payload data.

**Example Usage:**

```python
obj_type, data = read_object("a1b2c3d4e5f6...")
print(f"Object type: {obj_type}")
print(f"Object data: {data.decode('utf-8')}")
```

---

## Object Types: Blob, Tree, and Commit

Git uses three primary object types to represent repository data and history:

```
+-----------------+
|      Blob       |
| (file contents) |
+-----------------+
        |
        v
+-----------------+
|      Tree       |
| (directory listing) |
+-----------------+
        |
        v
+-----------------+
|     Commit      |
| (snapshot + metadata) |
+-----------------+
```

- **Blob:** Stores raw file data.  
- **Tree:** Represents directories; contains entries mapping filenames to blob or subtree hashes.  
- **Commit:** Contains metadata (author, date, message) and points to a tree representing the repository state.

---

## Encoding Pack Files

### `encode_pack(objects: List[str]) -> bytes`

**Purpose:**  
Encodes multiple objects into a single pack file for efficient storage and transfer.

**Parameters:**  
- `objects`: A list of SHA-1 hashes of objects to be packed.

**Process Overview:**  
- Objects are compressed and delta-encoded relative to similar objects to minimize size.  
- The pack file format includes a header, object data, and a checksum trailer.

**ASCII Diagram of Pack Structure:**

```
+-------------------+
|  Pack Header      |
+-------------------+
|  Object 1 Data    |
+-------------------+
|  Object 2 Data    |
+-------------------+
|     ...           |
+-------------------+
|  Pack Checksum    |
+-------------------+
```

**Example Usage:**

```python
pack_data = encode_pack([blob_hash, tree_hash, commit_hash])
with open('.git/objects/pack/pack-xxxx.pack', 'wb') as f:
    f.write(pack_data)
```

---

## Locating and Retrieving Objects Recursively

### `get_object_recursive(sha1_hash: str) -> Dict`

**Purpose:**  
Recursively retrieves an object and all referenced objects (e.g., trees and commits) from the database, reconstructing the repository state.

**Parameters:**  
- `sha1_hash`: The SHA-1 hash of the starting object (usually a commit).

**Operation:**  
1. Read the object using `read_object`.  
2. If the object is a commit, retrieve the referenced tree and parent commits recursively.  
3. If the object is a tree, retrieve all blobs and subtrees recursively.  
4. Return a nested dictionary representing the full object graph.

**Example Usage:**

```python
commit_hash = "d4e5f6a7b8c9..."
repository_state = get_object_recursive(commit_hash)
print(repository_state)
```

---

## Summary ASCII Diagram of Object Relationships

```
Commit (points to)
      |
      v
    Tree (directory)
    /           \
Blob(file)    Tree(subdirectory)
                  |
                  v
                Blob(file)
```

This hierarchical structure allows Git to efficiently represent snapshots of the repository at any given point in time.

---

By mastering these core concepts and functions, developers can confidently navigate and manipulate Git's object database, enabling advanced operations such as custom object storage, pack file creation, and deep repository inspection.
```