# init.md

# Repository Initialization and Setup  
## Instructions and details on initializing a new repository and creating necessary directories and files

---

## Overview

This document provides detailed instructions and explanations on initializing a new Git repository using the `pygit` implementation. It covers the creation of the repository directory structure, essential files, and initial setup procedures necessary for a functional Git repository. This file is part of the broader "Repository Initialization and Setup" section, helping users understand how to bootstrap a repository before performing further operations like committing, branching, or pushing.

---

## Function Documentation

### `init(repo)`

**Purpose:**  
Initialize a new Git repository by creating the necessary directory structure and essential files under the specified repository directory.

**Parameters:**  
- `repo` (str): The path or name of the repository directory to initialize.

**Operation Steps:**  
1. Create the root repository directory named `repo`.  
2. Inside `repo`, create the `.git` directory which holds all Git-specific data.  
3. Within `.git`, create the following subdirectories:
   - `objects` (stores Git objects like blobs, trees, commits)
   - `refs` (stores references to branches and tags)
   - `refs/heads` (specifically for branch heads)  
4. Create the `HEAD` file in `.git` with content pointing to the default branch reference (`ref: refs/heads/master`).  
5. Print a confirmation message indicating the repository has been initialized.

**Example Usage:**

```python
import pygit

# Initialize a new repository named "my_project"
pygit.init("my_project")
```

**Result:**  
```
initialized empty repository: my_project
```

**ASCII Diagram: Repository Directory Structure After Initialization**

```
my_project/
└── .git/
    ├── HEAD                 # Points to refs/heads/master
    ├── objects/             # Git objects storage
    └── refs/
        └── heads/           # Branch heads storage
```

---

### `write_file(path, data)`

**Purpose:**  
Write raw bytes data to a file at the specified path. Used internally to write Git metadata files or objects.

**Parameters:**  
- `path` (str): The file path to write to.  
- `data` (bytes): The data to write into the file.

**Operation Steps:**  
1. Open the file at `path` in binary write mode.  
2. Write the byte data to the file.  
3. Close the file automatically (context manager).

**Example Usage:**

```python
pygit.write_file(".git/description", b"This is my repository")
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash of a Git object given its content and type, optionally writing the compressed object to the Git object store.

**Parameters:**  
- `data` (bytes): The content of the Git object (e.g., file content for blobs).  
- `obj_type` (str): Type of the object (`"blob"`, `"tree"`, `"commit"`, etc.).  
- `write` (bool, optional): If True, write the object to `.git/objects`. Defaults to `True`.

**Operation Steps:**  
1. Construct the object header as `"<obj_type> <len(data)>\0"`.  
2. Concatenate the header and data to form the full object data.  
3. Compute the SHA-1 hash of the full data.  
4. If `write` is True and the object does not already exist in the object store:  
   - Create the subdirectory under `.git/objects` named by the first two characters of the SHA-1.  
   - Compress and write the full data to a file named after the remaining 38 characters of the hash.  
5. Return the hex string of the SHA-1 hash.

**Example Usage:**

```python
content = b"Hello World\n"
sha1_hash = pygit.hash_object(content, "blob")
print(f"Object SHA-1: {sha1_hash}")
```

---

### Summary Diagram: Repository Initialization Workflow

```
+-------------------+
|   pygit.init()    |
+---------+---------+
          |
          v
+-------------------+
| Create repo dir   |
+-------------------+
          |
          v
+-------------------+
| Create .git dir   |
+-------------------+
          |
          v
+-------------------+
| Create objects/   |
| Create refs/      |
| Create refs/heads/|
+-------------------+
          |
          v
+-------------------+
| Write HEAD file   |
+-------------------+
          |
          v
+-------------------+
| Initialization    |
| complete          |
+-------------------+
```

---

## Additional Notes

- This initialization establishes the basic file layout compatible with Git's internal structure, allowing subsequent commands such as `add`, `commit`, and `push` to function correctly.  
- The default branch is set to `master` by the `HEAD` file, which can be adjusted later if needed.  
- The `write_file` function is a utility used throughout the `pygit` codebase to create Git internal files and is critical for maintaining repository integrity.

---

This concludes the `init.md` documentation for repository initialization and setup. Users are encouraged to proceed with adding files and committing changes after this initial setup.