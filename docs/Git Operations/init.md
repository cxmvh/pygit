# pygit init Documentation

## Overview

The `init.md` file documents the `pygit.init` function responsible for initializing a new Git repository. This process includes creating the necessary directory structure and essential files within the `.git` directory to prepare the repository for further Git operations. This file is part of the **Getting Started** section of the pygit documentation, helping users set up their repository environment before performing tasks such as adding files, committing changes, or pushing to remotes. The initialization is a foundational step, ensuring that the repository layout adheres to Git standards and pygit's expectations.

---

## Function Documentation

### `init(repo)`

**Purpose:**  
Create a new directory for the repository and initialize the `.git` directory with the standard Git directory structure and files.

**Parameters:**  
- `repo` (str): The path or name of the directory where the new repository will be created.

**Description:**  
This function sets up the basic infrastructure of a Git repository:

1. Creates the main repository directory specified by `repo`.
2. Creates the `.git` subdirectory inside the repository, which houses all Git metadata.
3. Inside `.git`, creates mandatory subdirectories:
   - `objects` – for storing Git objects (blobs, trees, commits).
   - `refs` and `refs/heads` – for storing branch references.
4. Creates the `HEAD` file inside `.git`, which points to the default branch reference (`refs/heads/master`).

After successful initialization, it prints a confirmation message indicating the repository has been initialized.

**Code Example:**

```python
import os
from pygit import write_file  # Assuming write_file is available in pygit utilities

def init(repo):
    """Create directory for repo and initialize .git directory."""
    os.mkdir(repo)
    os.mkdir(os.path.join(repo, '.git'))
    for name in ['objects', 'refs', 'refs/heads']:
        os.mkdir(os.path.join(repo, '.git', name))
    write_file(os.path.join(repo, '.git', 'HEAD'), b'ref: refs/heads/master')
    print('initialized empty repository: {}'.format(repo))
```

**Usage Example:**

```python
# Initialize a new repository in the directory 'myrepo'
init('myrepo')
```

This creates the following directory structure inside `myrepo`:

```
myrepo/
└── .git/
    ├── HEAD
    ├── objects/
    └── refs/
        └── heads/
```

---

### `write_file(path, data)`

**Purpose:**  
Utility function to write binary data to a file at the specified path.

**Parameters:**  
- `path` (str): The file path where data should be written.  
- `data` (bytes): The binary data to write into the file.

**Description:**  
This helper function opens the file in binary write mode and writes the provided data. It is used by `init` to write the `HEAD` file content.

**Code Example:**

```python
def write_file(path, data):
    """Write data bytes to file at given path."""
    with open(path, 'wb') as f:
        f.write(data)
```

---

## Additional Context and Related Functions

The repository initialization is the first step before using functions such as:

- `add(paths)`: Add files to the git index.
- `commit(message, author=None)`: Commit changes to the repository.
- `push(git_url, username=None, password=None)`: Push commits to a remote repository.

These functions rely on the repository structure created by `init` to function correctly.

---

## ASCII Diagram: Repository Structure after `init(repo)`

```
<repo>/
└── .git/
    ├── HEAD                   # Points to the default branch
    ├── objects/               # Stores git objects (blobs, trees, commits)
    └── refs/
        └── heads/
            └── master         # Reference to the master branch commit (created later)
```

The `HEAD` file typically contains:

```
ref: refs/heads/master
```

indicating the current active branch.

---

# Summary

The `init` function is essential for bootstrapping a new Git repository with the expected layout and files, enabling further git operations within the pygit framework. It ensures that the `.git` directory and its subdirectories exist and creates the initial `HEAD` reference, signaling the starting point for commits and branch management.