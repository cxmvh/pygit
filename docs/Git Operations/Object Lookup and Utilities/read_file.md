# read_file.md

# Overview

The `read_file` module provides a simple utility function to read the contents of a file as bytes. This functionality is critical in the pygit project because many Git operations, such as reading object files, index files, and ref files, require loading raw byte data from the filesystem. The `read_file` function serves as a foundational building block used by higher-level functions like `read_object`, `get_local_master_hash`, and others that manipulate Git repository data.

Within the overall pygit documentation tree, `read_file.md` resides under the **Object Utilities** section in the **Object Handling** category. It supports core operations like reading Git objects (`read_object`) and displaying Git object contents (`cat_file`), and it is also utilized in remote operations like pushing commits (`push`).

---

# Functions

## read_file(path)

### Purpose

Reads the entire contents of the file located at the specified filesystem path and returns it as a bytes object.

This function is used whenever raw file data needs to be read, such as reading Git object files (which are zlib compressed), index files, or reference files within the `.git` directory.

### Parameters

- `path` (str): The filesystem path to the file to be read.

### Returns

- `bytes`: The raw byte contents of the file.

### Usage Details

- The function opens the file in binary mode (`'rb'`) to ensure that the data is returned exactly as stored, without any decoding or newline translation.
- It reads the entire file content at once and returns it.
- If the file does not exist, a `FileNotFoundError` will be raised, which should be handled by the caller if necessary.

### Example Usage

```python
from pygit import read_file

# Read the raw bytes of a git object file by its path
object_path = '.git/objects/ab/cdef1234567890abcdef1234567890abcdef12'
data_bytes = read_file(object_path)

print(f"Read {len(data_bytes)} bytes from object file.")
```

### Integration Example in pygit

The following snippet demonstrates how `read_file` is used within the `read_object` function to read and decompress Git object data:

```python
def read_object(sha1_prefix):
    """Read object with given SHA-1 prefix and return tuple of
    (object_type, data_bytes), or raise ValueError if not found.
    """
    path = find_object(sha1_prefix)
    full_data = zlib.decompress(read_file(path))
    nul_index = full_data.index(b'\x00')
    header = full_data[:nul_index]
    obj_type, size_str = header.decode().split()
    size = int(size_str)
    data = full_data[nul_index + 1:]
    assert size == len(data), 'expected size {}, got {} bytes'.format(
            size, len(data))
    return (obj_type, data)
```

---

# ASCII Diagram: File Reading Operation

```
+-------------------+
|   read_file(path)  |
+-------------------+
          |
          v
+---------------------------+
| Open file at 'path' in rb |
+---------------------------+
          |
          v
+-----------------------------+
| Read entire file content as |
| raw bytes                   |
+-----------------------------+
          |
          v
+-----------------+
| Return bytes    |
+-----------------+
```

This diagram shows the simple flow of the `read_file` function: opening a file in binary mode, reading all bytes, and returning them.

---

# Summary

The `read_file` function is a foundational utility in pygit for reading raw byte data from files. Its straightforward implementation ensures transparent and reliable file reading for all higher-level Git operations that depend on raw file data, including object parsing, index reading, and ref lookups.