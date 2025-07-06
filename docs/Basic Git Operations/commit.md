# Commit Process Documentation

## Overview

This document details the commit process within the Git implementation, focusing on the creation of commit objects and the subsequent update of the master branch reference. It is part of the **Basic Git Operations** section, providing essential insights into how changes staged in the index are permanently recorded in the repository history. Understanding this process is fundamental for developers working with the commit functionality (`pygit.commit`) and for those aiming to extend or troubleshoot commit-related features.

---

## Functions

### `commit`

#### Purpose

The `commit` function creates a new commit object encapsulating the current state of the repository as represented by the index. It then updates the `master` branch reference to point to this new commit, effectively advancing the repository history.

#### Parameters

- **message** (string): The commit message describing the changes.
- **repo_path** (string, optional): Path to the Git repository root; defaults to the current directory.
- **author** (string, optional): Author information in the format `"Name <email>"`.
- **parent** (string, optional): Hash of the parent commit; if omitted, the commit is treated as the initial commit.

#### Preconditions

- The index must be up to date with the desired changes staged.
- The repository must be initialized with a valid `.git` directory structure.
- The `master` branch reference exists or will be created if this is the initial commit.

#### Operation Steps

1. **Read the current index state:** The function reads the staging area to determine which files and their corresponding blob hashes will be included in the commit.

2. **Create a tree object:** Based on the index entries, it builds a tree object representing the snapshot of the repository at this commit.

3. **Compose the commit object:** The commit object includes:
    - The tree hash from step 2.
    - The parent commit hash (if any).
    - Author and committer metadata.
    - The commit message.
    - Commit timestamp and timezone.

4. **Hash and store the commit object:** The commit object is serialized and hashed using SHA-1, then stored in the Git object database.

5. **Update the master branch reference:** The `refs/heads/master` file is updated to point to the new commit hash, effectively moving the branch tip.

6. **Return the new commit hash:** This allows calling functions or users to track the commit created.

#### Example Usage

```python
from pygit import commit

# Commit staged changes with a message and author info
new_commit_hash = commit(
    message="Implement feature X with tests",
    repo_path="/path/to/repo",
    author="Alice Developer <alice@example.com>"
)

print(f"New commit created: {new_commit_hash}")
```

---

## ASCII Diagram of Commit Creation Flow

```
+------------------+
|    Index Files   |  <-- current staged changes
+--------+---------+
         |
         v
+------------------+
|   Create Tree    |  <-- snapshot of file structure and contents
+--------+---------+
         |
         v
+------------------+
|  Compose Commit  |  <-- metadata + tree + parent commit
+--------+---------+
         |
         v
+------------------+
| Store Commit Obj |  <-- object database (.git/objects)
+--------+---------+
         |
         v
+------------------+
| Update Master Ref|  <-- refs/heads/master updated to new commit hash
+------------------+
```

---

## See Also

- [repository_initialization.md](../Repository%20Initialization%20and%20Structure/repository_initialization.md): For details on setting up the repository structure.
- [index_management.md](../Index%20Management/index_management.md): To understand how files are staged in the index prior to committing.
- [object_storage.md](../Object%20Storage%20and%20Management/object_storage.md): For information on how Git stores objects like blobs, trees, and commits.

---

This documentation provides a comprehensive technical reference for the commit process in the codebase, facilitating easier maintenance, enhancement, and usage by developers.