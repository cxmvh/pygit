# cat_file.md

## Overview

This document provides detailed information about the `cat_file` command and related object reading functions within the `pygit` implementation. It focuses on how Git objects are read, located, and interpreted from the repository storage. The file explains core functions such as `read_object`, `read_tree`, and `find_object`, which together enable the inspection and retrieval of Git objects by their hashes. This document fits within the broader **Object Management** section of the repository documentation, complementing files like `objects.md` and `objects_and_packs.md` by focusing specifically on reading and displaying object contents.

---

## Functions

### `find_object(sha)`

**Purpose:**  
Locate a Git object in the repository by its SHA-1 hash. This function searches the object database to find the compressed object data corresponding to the given SHA.

**Parameters:**  
- `sha` (str): The SHA-1 hash string identifying the object to find.

**Description:**  
`find_object` takes a SHA-1 hash and looks up the corresponding object file in the `.git/objects` directory. Git stores objects in a two-level directory structure: the first two characters form a directory, and the remaining 38 characters form the filename. This function handles locating this file and returning its raw compressed contents.

**Operation Steps:**  
1. Extract the first two characters of the SHA as the directory name.  
2. Extract the remaining characters as the filename.  
3. Construct the full path to the object file.  
4. Read and decompress the file contents.  
5. Return the decompressed object data.

**Example Usage:**
```python
sha = "e69de29bb2d1d6434b8b29ae775ad8c2e48c5391"
object_data = find_object(sha)
print(object_data)
```

---

### `read_object(sha)`

**Purpose:**  
Read and parse a Git object from the repository by its SHA-1 hash.

**Parameters:**  
- `sha` (str): The SHA-1 hash string of the object to read.

**Returns:**  
- `(object_type, object_content)` tuple:
  - `object_type` (str): The type of the object (e.g., `commit`, `tree`, `blob`, `tag`).
  - `object_content` (bytes or dict): The raw content of the object or a parsed representation (for trees).

**Description:**  
This function uses `find_object` to locate the raw object data. It then parses the object header to identify the object type and size, extracts the content, and returns both. For `tree` objects, the content is typically parsed into structured entries representing directory entries.

**Operation Steps:**  
1. Call `find_object(sha)` to get the decompressed raw object data.  
2. Parse the header to separate the object type and size from content.  
3. Depending on the object type:  
   - For blobs and commits, return raw content.  
   - For trees, parse the entries into a list of objects representing files/directories.  
4. Return the `(object_type, object_content)`.

**Example Usage:**
```python
sha = "f572d396fae9206628714fb2ce00f72e94f2258f"
obj_type, content = read_object(sha)
print(f"Type: {obj_type}")
if obj_type == "blob":
    print(content.decode())
elif obj_type == "tree":
    for entry in content:
        print(f"{entry['mode']} {entry['path']} {entry['sha']}")
```

---

### `read_tree(tree_sha)`

**Purpose:**  
Read and parse a Git tree object, returning its contents as a list of directory entries.

**Parameters:**  
- `tree_sha` (str): The SHA-1 hash of the tree object.

**Returns:**  
- `List[dict]`: Each dictionary contains keys such as `mode`, `type`, `sha`, and `path` representing an entry in the tree.

**Description:**  
Trees represent directory snapshots in Git. This function reads the tree object using `read_object`, then parses each entry which includes the file mode, object type (blob or tree), SHA, and filename. This allows recursive exploration of repository contents.

**Operation Steps:**  
1. Use `read_object(tree_sha)` to get the tree data.  
2. Parse the binary tree content entries:  
   - Each entry includes mode, filename, null separator, and SHA in binary form.  
3. Convert each entry into a dictionary with readable fields.  
4. Return the list of entries.

**Example Usage:**
```python
tree_sha = "4b825dc642cb6eb9a060e54bf8d69288fbee4904"
entries = read_tree(tree_sha)
for entry in entries:
    print(f"{entry['mode']} {entry['type']} {entry['sha']} {entry['path']}")
```

---

### `cat_file(sha, fmt='pretty')`

**Purpose:**  
Display the contents of a Git object in a human-readable or raw format, mimicking the behavior of the `git cat-file` command.

**Parameters:**  
- `sha` (str): The SHA-1 hash of the object to display.  
- `fmt` (str, optional): Output format, e.g., `'pretty'` (default) for formatted display or `'raw'` for raw bytes.

**Description:**  
This function retrieves the Git object specified by `sha` and prints its contents. It supports multiple output formats:  
- `pretty`: prints a formatted, human-readable representation of the object (e.g., commit message, tree listing).  
- `raw`: prints the raw binary content of the object.

It uses `read_object` to get the object type and content, then processes the output accordingly.

**Operation Steps:**  
1. Call `read_object(sha)` to get the object type and content.  
2. If `fmt` is `raw`, output the raw content as bytes.  
3. If `fmt` is `pretty`, format the output based on the object type:  
   - For `commit`, print commit metadata and message.  
   - For `tree`, list all entries in a readable way.  
   - For `blob`, print the text content.  
   - For other types, print a generic message or raw content.  
4. Display the formatted output.

**Example Usage:**
```python
sha = "f572d396fae9206628714fb2ce00f72e94f2258f"
cat_file(sha, fmt='pretty')
```

---

## ASCII Diagram: Git Object Storage Structure

```
.git/objects/
├── 4b/
│   └── 825dc642cb6eb9a060e54bf8d69288fbee4904  (object file)
├── e6/
│   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
└── f5/
    └── 72d396fae9206628714fb2ce00f72e94f2258f
```

- The first two characters of the SHA form the directory name.  
- The remaining 38 characters form the object filename.  
- `find_object` locates these files to read object data.

---

This concludes the detailed reference for the `cat_file` command and related object reading functions in `pygit`. For additional context on object creation, encoding, and packfiles, see the neighboring documents [objects.md](./objects.md) and [objects_and_packs.md](./objects_and_packs.md).