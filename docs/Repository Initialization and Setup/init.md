# init.md

# Guide on Creating and Initializing a New Git Repository Directory Structure

---

## Overview

This document provides a step-by-step guide on how to create and initialize a new Git repository directory structure using the `init` function from the `pygit` module. It covers the creation of the necessary directory hierarchy and essential files inside the `.git` folder to set up an empty Git repository ready for further operations. This guide fits within the broader "Repository Initialization and Setup" section of the documentation tree, which focuses on setting up the foundational `.git` directory and configuring the repository for version control.

Initializing a repository is the first critical step before tracking changes, committing objects, and managing branches. Understanding the directory layout and initialization process is fundamental for users and developers working with or extending `pygit`.

---

## Important Functions

### Function: `init(repo)`

#### Purpose

Creates a new directory for the repository and initializes the `.git` directory structure inside it. This sets up the standard directories and files required for a Git repository to function, such as `objects`, `refs`, and `refs/heads`, along with the `HEAD` file pointing to the default branch.

#### Parameters

- `repo` (str): The path where the new Git repository directory should be created.

#### Preconditions

- The specified `repo` directory path must not already exist.
- The executing user must have permission to create directories and files at the specified location.

#### Operation Steps

1. Create the main repository directory specified by `repo`.
2. Inside `repo`, create a `.git` directory to hold Git metadata.
3. Within `.git`, create subdirectories:
   - `objects` for object storage (blobs, trees, commits).
   - `refs` for references.
   - `refs/heads` for branch heads.
4. Write the `HEAD` file inside `.git`, with contents pointing to `refs/heads/master`, indicating the default branch.
5. Print a confirmation message indicating successful initialization.

#### Usage Example

```python
import pygit

# Initialize a new Git repository in directory 'my_project'
pygit.init('my_project')

# Expected Output:
# initialized empty repository: my_project
```

#### ASCII Diagram: Repository Directory Layout After Initialization

```
my_project/
└── .git/
    ├── objects/
    ├── refs/
    │   └── heads/
    └── HEAD  (contains: 'ref: refs/heads/master')
```

---

### Function: `write_file(path, data)`

#### Purpose

Helper function to write byte data to a specified file path, creating or overwriting the file as needed.

#### Parameters

- `path` (str): The file path to write data to.
- `data` (bytes): The byte content to write to the file.

#### Operation Steps

1. Open the file at `path` in binary write mode (`'wb'`).
2. Write the `data` bytes to the file.
3. Close the file automatically upon exiting the context.

#### Usage Example

```python
import pygit

# Write the string 'Hello, Git!' as bytes to a file
pygit.write_file('my_project/.git/README', b'Hello, Git!')
```

---

### Additional Context: Related Repository Initialization Functions

Although the `init` function sets up the repository structure, further operations like creating commits and managing the index depend on other functions documented elsewhere, such as:

- `hash_object(data, obj_type, write=True)`: Hashes and stores objects in `.git/objects`.
- `write_index(entries)`: Writes entries to the Git index file.
- `commit(message, author=None)`: Creates a commit in the repository.
- `get_local_master_hash()`: Retrieves the current commit hash of the master branch.

These functions rely on the repository structure created by `init`.

---

## Summary

The `init` function is the entry point for establishing a new Git repository in a directory. It creates the `.git` directory and essential subdirectories and files, enabling the repository to track content and changes. This initialization is a prerequisite for any further Git operations within the repository.

---

# Code Reference: `init` Function

```python
def init(repo):
    """Create directory for repo and initialize .git directory."""
    import os
    os.mkdir(repo)
    os.mkdir(os.path.join(repo, '.git'))
    for name in ['objects', 'refs', 'refs/heads']:
        os.mkdir(os.path.join(repo, '.git', name))
    write_file(os.path.join(repo, '.git', 'HEAD'), b'ref: refs/heads/master')
    print('initialized empty repository: {}'.format(repo))
```

---

# Appendix: `write_file` Function Reference

```python
def write_file(path, data):
    """Write data bytes to file at given path."""
    with open(path, 'wb') as f:
        f.write(data)
```

---

This concludes the documentation for initializing a new Git repository directory structure using `pygit.init`. For further repository operations such as adding files, committing changes, and inspecting status, refer to the related documentation files within the "Repository Initialization and Setup" and other related sections.