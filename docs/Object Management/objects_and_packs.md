# objects_and_packs.md

## Overview

This document covers the internal mechanisms of Git object storage and packfile handling within the pygit project. It elaborates on how Git objects are encoded, hashed, stored, and efficiently packed into packfiles for optimized storage and transfer. The key functions discussed include `encode_pack_object`, responsible for encoding individual objects for packfiles, and `create_pack`, which assembles multiple Git objects into a single packfile. This file sits under the **Object Management** section of the documentation tree, complementing related files like `objects.md` and `cat_file.md` by providing deeper insights into packfile creation and low-level object encoding. Understanding these processes is crucial for developers working on object serialization, storage optimization, or network transfer features such as pushing to remote repositories.

---

## Function Documentation

### `encode_pack_object`

#### Purpose

Encodes a single Git object into the packfile format, preparing it for inclusion in a packfile. This encoding compresses the object data and formats it according to Git's packfile specification, enabling efficient storage and transfer.

#### Parameters

- `obj_type` (str): The type of the Git object (e.g., `"blob"`, `"tree"`, `"commit"`).
- `obj_data` (bytes): The raw data payload of the Git object.
- `delta_base` (bytes or None): Optional base object data if encoding a delta object (for delta compression).
  
#### Operation

1. **Header Encoding:**  
   The function starts by encoding the object type and size into a variable-length header format defined by Git's packfile specification.

2. **Compression:**  
   The raw object data or delta data is compressed using zlib compression to reduce size.

3. **Delta Handling (Optional):**  
   If `delta_base` is provided, the function computes the delta between `obj_data` and `delta_base` and encodes the delta instructions instead of the full object.

4. **Return Format:**  
   Returns the fully encoded packfile object as bytes, ready to be written into a packfile.

#### Example Usage

```python
obj_type = "blob"
obj_data = b"Hello, Git object content!"
encoded_object = encode_pack_object(obj_type, obj_data, delta_base=None)

with open("example.pack", "ab") as packfile:
    packfile.write(encoded_object)
```

---

### `create_pack`

#### Purpose

Creates a Git packfile from a collection of Git objects, encoding each object and assembling them into a single file along with a packfile index. Packfiles are used to efficiently store and transfer multiple objects in compressed form.

#### Parameters

- `objects` (list of tuples): Each tuple contains `(obj_type, obj_data)` representing the type and raw data of a Git object to be included.
- `packfile_path` (str): The file path where the generated packfile will be saved.
- `index_path` (str): The file path where the corresponding packfile index (.idx) will be saved.

#### Operation

1. **Packfile Header:**  
   Writes the packfile signature and version number, followed by the count of objects.

2. **Object Encoding:**  
   Iterates over the list of objects, encoding each via `encode_pack_object`. Optionally, delta compression can be applied to reduce packfile size.

3. **Object Data Writing:**  
   Writes each encoded object sequentially into the packfile.

4. **Checksum Calculation:**  
   Computes a SHA-1 checksum of all the packfile data written so far and appends it to the end of the packfile.

5. **Index Creation:**  
   Builds a packfile index file that maps object hashes to their offsets within the packfile, allowing fast object lookup.

6. **File Writing:**  
   Saves the packfile and index files to disk at the specified paths.

#### Example Usage

```python
objects = [
    ("blob", b"First blob content"),
    ("commit", b"commit object content here"),
    ("tree", b"tree object content")
]

packfile_path = "./.git/objects/pack/pack-123456.pack"
index_path = "./.git/objects/pack/pack-123456.idx"

create_pack(objects, packfile_path, index_path)
```

---

### `hash_object_data`

#### Purpose

Computes the SHA-1 hash of a Git object’s data combined with its type and size header. This hash serves as the unique identifier for the object in Git's object database.

#### Parameters

- `obj_type` (str): The Git object type (e.g., `"blob"`, `"tree"`, `"commit"`).
- `obj_data` (bytes): The raw data content of the object.

#### Operation

1. **Header Construction:**  
   Constructs a header string in the format:  
   `"{obj_type} {size}\0"`  
   where `size` is the length of `obj_data`.

2. **Concatenation:**  
   Concatenates the header and the raw data bytes.

3. **SHA-1 Hashing:**  
   Applies SHA-1 hashing to the concatenated bytes to produce the object’s hash.

4. **Return:**  
   Returns the SHA-1 hash as a hexadecimal string.

#### Example Usage

```python
obj_type = "blob"
obj_data = b"Sample data for hashing"
obj_hash = hash_object_data(obj_type, obj_data)

print(f"Object SHA-1 hash: {obj_hash}")
# Output: Object SHA-1 hash: e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
```

---

## ASCII Diagram: Packfile Structure

Below is a simplified representation of a Git packfile structure to illustrate how encoded objects are organized within the packfile:

```
+---------------------------+
| Packfile Header           |
|  - Signature: "PACK"      |
|  - Version number         |
|  - Number of objects (N)  |
+---------------------------+
| Encoded Object 1          |
|  - Type and size header   |
|  - Compressed data        |
+---------------------------+
| Encoded Object 2          |
|  ...                     |
+---------------------------+
| ...                       |
+---------------------------+
| Encoded Object N          |
+---------------------------+
| SHA-1 checksum of packfile|
+---------------------------+
```

Each encoded object is produced by the `encode_pack_object` function, and the entire packfile is assembled via `create_pack`.

---

## See Also

- [`objects.md`](./objects.md) — Covers basic object creation, reading, and hashing.
- [`cat_file.md`](./cat_file.md) — Details commands and functions for reading Git objects.
- [`push.md`](./push.md) — Explains pushing objects and packfiles to remote repositories.

---

This documentation equips developers with the knowledge to understand and extend Git object storage and packing mechanisms within pygit, crucial for repository performance and network operations.