# init.md

# Repository Initialization and Core Objects  
## Instructions and Details about Initializing a Git Repository and Setting up its Directory Structure

---

## Overview

This document provides a comprehensive guide to initializing a new Git repository using the `pygit.init` functionality. It covers the steps required to create the repository directory, set up the essential `.git` directory structure, and prepare the repository for version control operations. This initialization process is fundamental as it lays the groundwork for subsequent operations like adding files, committing changes, and managing objects within the repository.

Within the broader documentation tree, this file is part of the **Repository Initialization and Core Objects** section, which collectively documents the foundational steps and core mechanisms behind repository setup and object management in the `pygit` project.

---

## Functions for Repository Initialization and Setup

### 1. `init(repo)`

#### Purpose
Initialize a new Git repository by creating the specified directory and setting up the minimal `.git` directory structure required for Git operations.

#### Parameters
- `repo` (str): The path or name of the directory where the new repository will be initialized.

#### Description
The `init` function performs the following steps in order:

1. Creates the top-level repository directory specified by `repo`.
2. Creates the `.git` directory inside the repository to hold Git metadata.
3. Inside `.git`, creates essential subdirectories: `objects`, `refs`, and `refs/heads`.
4. Writes the initial `HEAD` file pointing to the default branch `refs/heads/master`.
5. Prints a confirmation message indicating the repository has been initialized.

#### Step-by-Step Operation

```
repo/                      # Repository directory created
└── .git/                  # Git metadata directory
    ├── objects/           # Storage for Git objects (blobs, trees, commits)
    ├── refs/              # References directory
    │   └── heads/         # Branch heads directory
    └── HEAD               # Points to the current branch (master by default)
```

1. `os.mkdir(repo)` creates the repository directory.
2. `os.mkdir(os.path.join(repo, '.git'))` creates the `.git` directory.
3. For each directory in [`objects`, `refs`, `refs/heads`], create the corresponding subdirectory inside `.git`.
4. Write the string `ref: refs/heads/master` into `.git/HEAD` to set the default branch.
5. Print a message confirming initialization.

#### Example Usage

```python
import pygit

# Initialize a new git repository at './myproject'
pygit.init('myproject')
```

Expected output:

```
initialized empty repository: myproject
```

---

### 2. `write_file(path, data)`

#### Purpose
Write binary data to a file at the specified path.

#### Parameters
- `path` (str): File system path where data will be written.
- `data` (bytes): Byte sequence to write to the file.

#### Description
This utility function opens the file at `path` in binary write mode and writes the given `data` bytes. It is used internally during repository initialization to write files like `.git/HEAD`.

#### Example Usage

```python
# Write some bytes to a file
pygit.write_file('myproject/.git/HEAD', b'ref: refs/heads/master')
```

---

### Supporting ASCII Diagram for Repository Initialization

```
+-------------------+
| repo/             |  <-- Repository root directory
| +-- .git/         |  <-- Git metadata directory
|     +-- objects/  |  <-- Stores all git objects (blobs, trees, commits)
|     +-- refs/     |  <-- Stores references (branches, tags)
|         +-- heads/|  <-- Branch heads (e.g., master)
|     +-- HEAD      |  <-- Points to the current branch reference
+-------------------+
```

---

## Summary

The `init` function is the critical first step for creating a Git repository. It sets up the canonical `.git` directory structure that Git expects, enabling subsequent commands to operate correctly. The helper function `write_file` facilitates writing necessary files during this setup. Together, these form the foundation of repository management in the `pygit` library.

---

## Related Functions in Repository Initialization

While `init` focuses on setting up the repository structure, other important functions interacting closely with the initialized repo include:

- `hash_object(data, obj_type, write=True)`: Hashes data, optionally writing Git objects to the `objects` directory.
- `write_index(entries)`: Writes the Git index file based on entries.
- `commit(message, author=None)`: Commits changes to the repository.
- `add(paths)`: Adds files to the Git index.

These functions build upon the initial setup performed by `init` to manage content and history in the repository.

---

This concludes the documentation for `init.md` covering Git repository initialization and directory setup.