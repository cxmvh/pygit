# init.md

# Initialization of a New Git Repository and Directory Structure

---

## Overview

This document covers the initialization process of a new Git repository using the `pygit` implementation. It details how a new repository directory and its essential `.git` subdirectory structure are created and prepared for version control operations. This file fits within the **Repository Initialization and Management** section of the broader documentation tree, which addresses repository setup, state inspection, and core repository management functions.

The initialization step is fundamental as it sets up the groundwork for all subsequent Git operations by creating the necessary directory layout and files such as the object store and references. Understanding this process is key for developers integrating or extending the `pygit` tool or for those interested in the inner workings of Git repository management.

---

## Function Documentation

### `init(repo)`

#### Purpose

Create a new directory for the repository named `repo` and initialize the `.git` directory structure inside it. This includes creating subdirectories for Git objects, references, and heads, and setting the default branch to `master`.

#### Parameters

- `repo` (str): The path or name of the directory where the new Git repository will be initialized.

#### Preconditions

- The directory specified by `repo` must **not** already exist, as the function attempts to create it.
- The user must have write permissions to create directories and files at the specified location.

#### Operation Details

1. Create the main repository directory at the path given by `repo`.
2. Inside the repository directory, create a `.git` directory which holds all Git-specific data.
3. Within `.git`, create the following subdirectories:
   - `objects/` – stores all Git objects (blobs, trees, commits).
   - `refs/` – stores references to commit objects.
   - `refs/heads/` – stores branch references.
4. Create a `HEAD` file inside `.git` that points to the default branch `refs/heads/master`.
5. Print a confirmation message indicating successful initialization.

#### Usage Example

```python
import pygit

# Initialize a new git repository in directory 'myproject'
pygit.init('myproject')
```

Upon successful execution, the following directory structure will be created:

```
myproject/
└── .git/
    ├── objects/
    ├── refs/
    │   └── heads/
    └── HEAD
```

The content of `.git/HEAD` will be:

```
ref: refs/heads/master
```

This indicates that the default branch is set to `master`.

#### Source Code Snippet

```python
def init(repo):
    """Create directory for repo and initialize .git directory."""
    os.mkdir(repo)
    os.mkdir(os.path.join(repo, '.git'))
    for name in ['objects', 'refs', 'refs/heads']:
        os.mkdir(os.path.join(repo, '.git', name))
    write_file(os.path.join(repo, '.git', 'HEAD'), b'ref: refs/heads/master')
    print('initialized empty repository: {}'.format(repo))
```

---

### `write_file(path, data)`

#### Purpose

Write raw byte data to a file at the specified path. This utility function is used internally to create files during repository initialization and other operations.

#### Parameters

- `path` (str): The file system path where the data should be written.
- `data` (bytes): The data to write to the file.

#### Operation Details

1. Open the file at `path` in binary write mode.
2. Write the bytes `data` to the file.
3. Close the file automatically upon completion.

#### Usage Example

```python
pygit.write_file('.git/HEAD', b'ref: refs/heads/master')
```

---

## ASCII Diagram: Repository Initialization Directory Structure

```
repo/               # Newly created repository directory
└── .git/           # Git metadata directory
    ├── objects/    # Stores Git objects (compressed blobs, trees, commits)
    ├── refs/       # Stores refs to commits and branches
    │   └── heads/  # Stores branch heads (e.g., master)
    └── HEAD       # File pointing to the current branch reference
```

---

## Related Functions and Workflow Context

While the `init` function sets up the repository structure, other functions in `pygit` leverage this initialization to manage repository content:

- `hash_object(data, obj_type, write=True)`: Hashes and stores objects in the `.git/objects` directory.
- `write_index(entries)`: Writes the Git index file to track the staging area.
- `commit(message, author=None)`: Creates commit objects referencing trees and parent commits.
- `get_status()`, `add(paths)`, `status()`, `diff()` and others that operate on the repository once initialized.

The successful execution of `init` is the prerequisite for all these operations.

---

# Summary

The `init.md` document provides a clear explanation of how a new Git repository is created at the file system level using the `pygit` toolkit. It focuses on the `init(repo)` function that initializes the `.git` directory and its essential subdirectories and files, enabling the repository to manage version control objects and references. This setup is crucial for the repository’s lifecycle and underpins all higher-level Git operations documented elsewhere.