# init.md

# How to Initialize a New Git Repository with pygit

---

## Overview

This document explains how to create and initialize a new Git repository using the `pygit` library. It focuses on the `pygit.init` function, which sets up the directory structure and essential files required for a Git repository. This file is part of the "Repository Initialization and Object Management" section of the documentation tree, which covers repository setup, object handling, and file management in pygit.

The initialization process prepares the `.git` directory with the necessary subdirectories (`objects`, `refs`, and `refs/heads`) and the `HEAD` file, pointing the repository to the default branch (`master`). This is the foundation on which other Git operations (commits, branches, pushes) are built.

---

## Function Documentation

### `init(repo)`

#### Purpose

Creates a new directory for the repository and initializes the `.git` directory structure inside it. This includes creating the essential subdirectories and the `HEAD` file, which points to the `master` branch by default.

#### Parameters

- `repo` (str): The path to the directory where the Git repository should be initialized. If the directory does not exist, it will be created.

#### Preconditions

- The directory specified by `repo` must not already exist (to avoid conflicts).
- The user has permissions to create directories and files at the specified location.

#### Description

1. Creates the main repository directory.
2. Creates the `.git` directory inside the repository directory.
3. Inside `.git`, creates the subdirectories:
   - `objects` — stores Git objects (blobs, trees, commits).
   - `refs` — holds references to branches and tags.
   - `refs/heads` — stores heads for local branches.
4. Writes the `HEAD` file pointing to the default branch `refs/heads/master`.
5. Prints a confirmation message indicating successful initialization.

#### Step-by-step operation

```plaintext
repo/                       # New repository directory
└── .git/                   # Git metadata directory
    ├── objects/            # Object storage directory
    ├── refs/               # References directory
    │   └── heads/          # Branch heads directory
    └── HEAD               # File pointing to current branch reference
```

The function performs:

```
mkdir(repo)
mkdir(repo/.git)
mkdir(repo/.git/objects)
mkdir(repo/.git/refs)
mkdir(repo/.git/refs/heads)
write_file(repo/.git/HEAD, b'ref: refs/heads/master')
```

#### Example Usage

```python
import pygit

# Initialize a new git repository at './myproject'
pygit.init('myproject')
```

**Output:**

```
initialized empty repository: myproject
```

---

### `write_file(path, data)`

#### Purpose

Writes raw byte data to a file at the specified path. Used internally to write repository files such as `HEAD` and object data.

#### Parameters

- `path` (str): Full file path where the data should be written.
- `data` (bytes): Byte content to write to the file.

#### Description

Opens the file in binary write mode and writes the provided bytes. Overwrites existing content if the file exists.

#### Example Usage

```python
pygit.write_file('myproject/.git/HEAD', b'ref: refs/heads/master')
```

---

## Additional Context: Repository Initialization Flow

The initialization process is a prerequisite for other pygit operations such as adding files, committing, and pushing changes. Once the repository is initialized, you can add files to the index, create commits, and manage branches.

Below is a simplified diagram showing the directory structure created by `init`:

```
myproject/
└── .git/
    ├── objects/
    ├── refs/
    │   └── heads/
    └── HEAD
```

- The `objects/` directory will eventually store compressed Git objects.
- The `refs/heads/` directory contains files for each branch (initially `master`).
- The `HEAD` file points to the current branch reference.

---

## Related Functions (Brief)

While `init` sets up the repository, several related functions are crucial for working with the repository:

- `hash_object(data, obj_type, write=True)`: Hashes and optionally writes Git objects to the object store.
- `write_index(entries)`: Writes the Git index file.
- `add(paths)`: Adds files to the Git index.
- `commit(message, author=None)`: Creates a commit object from the current index.
- `get_status()`: Retrieves the status of working copy files.
- `status()`: Prints the status of the working copy.
- `write_tree()`: Creates a tree object from the index entries.

Refer to their respective documentation files for detailed explanations and examples.

---

## Summary

The `pygit.init` function is the fundamental starting point for creating a new Git repository using pygit. It establishes the directory structure and configuration necessary for all subsequent Git operations. Understanding this function is essential for working with the pygit library's repository management features.