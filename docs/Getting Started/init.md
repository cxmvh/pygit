# pygit Initialization Documentation

## Overview

This document details the process of initializing a new Git repository using the `pygit` library's `init` function. It is part of the "Getting Started" section of the overall pygit documentation, which covers repository setup and basic file operations. The `init` function sets up the necessary directory structure and essential files required for a Git repository to function. This foundational step enables users to start tracking files, committing changes, and performing other Git operations within the newly created repository.

The initialization process creates a `.git` directory with subdirectories for objects and references, and sets the default branch to `master`. The documentation also references related functions such as `write_file` used during initialization to write the `HEAD` file, and subsequent operations like `push` that rely on a properly initialized repository.

---

## Function Documentation

### `init(repo)`

**Purpose:**

Initializes a new Git repository in the specified directory path `repo`. This involves creating the directory structure under `repo` and setting up the `.git` directory with essential subdirectories and files. After running this function, the repository is ready to track files and commits.

**Parameters:**

- `repo` (str): The file system path where the new repository will be created. This directory should not exist beforehand; the function will create it.

**Operation Steps:**

1. Create the root directory for the repository at the path specified by `repo`.
2. Within this directory, create the `.git` directory which stores all Git metadata.
3. Inside `.git`, create subdirectories:
   - `objects` — to store Git objects (blobs, trees, commits).
   - `refs` — to store references to commits.
   - `refs/heads` — to store branch heads.
4. Create the `HEAD` file inside `.git`, pointing it to the default branch `refs/heads/master`.
5. Print a confirmation message indicating successful initialization.

**Code Snippet:**

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

**Usage Example:**

```python
import pygit

# Initialize a new repository in ./myrepo
pygit.init('myrepo')

# Output:
# initialized empty repository: myrepo
```

This will create the following directory structure:

```
myrepo/
└── .git/
    ├── objects/
    ├── refs/
    │   └── heads/
    └── HEAD
```

---

### `write_file(path, data)`

**Purpose:**

Writes raw bytes data to a file at the specified path. Used internally by `init` to write the `HEAD` file.

**Parameters:**

- `path` (str): File path where data will be written.
- `data` (bytes): Data to write to the file.

**Operation Steps:**

1. Open the file at `path` in binary write mode.
2. Write the `data` bytes to the file.
3. Close the file.

**Code Snippet:**

```python
def write_file(path, data):
    """Write data bytes to file at given path."""
    with open(path, 'wb') as f:
        f.write(data)
```

**Usage Example:**

```python
write_file('myrepo/.git/HEAD', b'ref: refs/heads/master')
```

---

## ASCII Diagram: Repository Initialization Structure

```
<repo>/
└── .git/
    ├── objects/       # Stores Git objects (blobs, trees, commits)
    ├── refs/          # References to commits
    │   └── heads/     # Branch heads (e.g., master)
    └── HEAD           # Points to current branch (default: refs/heads/master)
```

---

## Related Functions in Initialization Context

- **`hash_object(data, obj_type, write=True)`**: Used to hash and optionally store objects in the `objects` directory.
- **`write_index(entries)`**: To write the Git index file after adding files.
- **`commit(message, author=None)`**: To create commits in the initialized repository.
- **`get_status()`**: To check the working directory status after initialization.
- **`add(paths)`**: To add files to the Git index.
- **`push(git_url, username=None, password=None)`**: To push commits to remote repositories (requires repository initialization first).

---

## Summary

The `init` function is the essential starting point for using `pygit`. It sets up the bare minimum Git structure, enabling all further Git operations to function correctly. Users should run this function first before adding files, committing, or pushing changes.

For more information on file operations after initialization, see related documents such as `file_operations.md`. For pushing changes to remote repositories, refer to `push.md`.

---

*End of `init.md` documentation.*