# Repository Initialization (`init.md`)

## Overview

This document provides detailed information on initializing a new Git repository using the `pygit` implementation. It covers the creation of the repository directory structure, including the essential `.git` subdirectories, and the setup of the initial HEAD reference. This file is part of the **Repository Initialization** section, which focuses on setting up a Git repository from scratch. Understanding this process is fundamental for developers working on repository creation and management in the `pygit` system.

---

## Function Documentation

### `init(repo)`

#### Purpose

Initializes a new Git repository in the specified directory by creating the necessary directory structure and setting the initial HEAD reference to the `master` branch.

#### Parameters

- `repo` (str): The path where the new repository directory will be created.

#### Operation

1. Creates the main repository directory specified by `repo`.
2. Creates a `.git` directory inside the repository to hold Git's internal data.
3. Within `.git`, creates the following subdirectories:
   - `objects`: Stores Git objects (blobs, trees, commits).
   - `refs`: Top-level directory for references.
   - `refs/heads`: Stores branch heads.
4. Creates a `HEAD` file within `.git` that points to the `master` branch by writing the reference string: `ref: refs/heads/master`.
5. Prints a confirmation message indicating successful repository initialization.

#### Example Usage

```python
import pygit

# Initialize a new Git repository in the directory 'my_project'
pygit.init('my_project')
```

**Expected Output:**

```
initialized empty repository: my_project
```

#### ASCII Diagram: Directory Structure After Initialization

```
my_project/
└── .git/
    ├── objects/
    ├── refs/
    │   └── heads/
    └── HEAD  (contains: "ref: refs/heads/master")
```

---

### `write_file(path, data)`

#### Purpose

Writes binary data to a file at the specified path. This is a utility function used during repository initialization to create and populate files such as `HEAD`.

#### Parameters

- `path` (str): The file path where data will be written.
- `data` (bytes): The binary data to write into the file.

#### Operation

- Opens the file at `path` in binary write mode.
- Writes the given `data` bytes to the file.
- Closes the file.

#### Example Usage

```python
from pygit import write_file

# Write the HEAD reference data
write_file('my_project/.git/HEAD', b'ref: refs/heads/master')
```

---

## Additional Context

While this document focuses on repository initialization details, it is closely related to other parts of the `pygit` system, such as:

- **Object Handling** (`objects.md`): Manages Git objects stored in the `objects` directory created during initialization.
- **Index Management** (`index.md`): Deals with staging and indexing files post-initialization.
- **Commit Management** (`commit.md`): Handles commit creation once the repository is initialized.

For detailed exploration of these functionalities, refer to their respective documentation files.

---

## Summary

The `init` function is the entry point to creating a new Git repository, establishing the standard Git directory layout and setting the initial branch reference. The `write_file` utility underpins this by enabling file creation and data writing, crucial for initializing repository metadata. Together, these functions establish the foundation from which full Git operations can proceed.