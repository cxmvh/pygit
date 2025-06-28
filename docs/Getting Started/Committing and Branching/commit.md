# commit.md

## Overview

This document explains how commits are created and managed in the `pygit` repository, focusing on the two primary functions: `commit` and `get_local_master_hash`. It is part of the "Committing and Branching" section of the overall pygit documentation, which covers processes related to committing changes and branch management. This file details the mechanisms by which the current index state is captured into a commit object, how the commit references are updated, and how the current commit hash of the local master branch is retrieved. Understanding these functions is essential for grasping how pygit records snapshots of the working directory and maintains the master branch's history.

---

## Functions

### `commit(message, author=None)`

#### Purpose

Creates a new commit object representing the current state of the index and updates the local `master` branch to point to this commit. It returns the SHA-1 hash of the created commit object.

#### Parameters

- `message` (str): The commit message describing the changes.
- `author` (str, optional): The author of the commit in the format `"Name <email>"`. If not provided, it is inferred from environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

#### Preconditions

- The repository must be initialized (`.git` directory exists).
- The index must be properly populated with the current staged changes.
- Environment variables for the author name and email should be set if `author` is not provided.

#### Operation Steps

1. Calls `write_tree()` to write the current index state as a tree object and get its SHA-1 hash.
2. Retrieves the current commit hash of the local master branch via `get_local_master_hash()` to use as the parent commit.
3. If no `author` is provided, constructs it from the `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL` environment variables.
4. Generates a timestamp for the commit and formats it with the UTC offset.
5. Constructs the commit object data, including:
    - Tree hash.
    - Parent commit hash (if any).
    - Author and committer information with timestamp.
    - Commit message.
6. Hashes the commit data using `hash_object` with type `"commit"`, which stores the commit object in the object database.
7. Updates the `refs/heads/master` file with the new commit hash.
8. Prints a confirmation message with the commit hash.
9. Returns the commit SHA-1 hash as a string.

#### Example Usage

```python
commit_hash = commit("Initial commit")
print(f"Created commit: {commit_hash}")
```

#### Code Snippet

```python
def commit(message, author=None):
    """Commit the current state of the index to master with given message.
    Return hash of commit object.
    """
    tree = write_tree()
    parent = get_local_master_hash()
    if author is None:
        author = '{} <{}>'.format(
                os.environ['GIT_AUTHOR_NAME'], os.environ['GIT_AUTHOR_EMAIL'])
    timestamp = int(time.mktime(time.localtime()))
    utc_offset = -time.timezone
    author_time = '{} {}{:02}{:02}'.format(
            timestamp,
            '+' if utc_offset > 0 else '-',
            abs(utc_offset) // 3600,
            (abs(utc_offset) // 60) % 60)
    lines = ['tree ' + tree]
    if parent:
        lines.append('parent ' + parent)
    lines.append('author {} {}'.format(author, author_time))
    lines.append('committer {} {}'.format(author, author_time))
    lines.append('')
    lines.append(message)
    lines.append('')
    data = '\n'.join(lines).encode()
    sha1 = hash_object(data, 'commit')
    master_path = os.path.join('.git', 'refs', 'heads', 'master')
    write_file(master_path, (sha1 + '\n').encode())
    print('committed to master: {:7}'.format(sha1))
    return sha1
```

---

### `get_local_master_hash()`

#### Purpose

Retrieves the current commit hash (SHA-1 string) referenced by the local `master` branch. This hash represents the latest commit on the local master branch.

#### Parameters

None.

#### Preconditions

- The repository must be initialized.
- The `refs/heads/master` file should exist if there are commits.

#### Operation Steps

1. Reads the contents of `.git/refs/heads/master`.
2. Decodes the contents from bytes to a UTF-8 string and trims whitespace.
3. Returns the commit hash as a string.
4. If the reference file does not exist (e.g., no commits yet), returns `None`.

#### Example Usage

```python
current_commit = get_local_master_hash()
if current_commit:
    print(f"Current master commit hash: {current_commit}")
else:
    print("No commits yet on master.")
```

#### Code Snippet

```python
def get_local_master_hash():
    """Get current commit hash (SHA-1 string) of local master branch."""
    master_path = os.path.join('.git', 'refs', 'heads', 'master')
    try:
        return read_file(master_path).decode().strip()
    except FileNotFoundError:
        return None
```

---

## Additional Context and Flow

The commit creation process involves several other functions:

- `write_tree()` reads the current index entries and creates a tree object representing the directory snapshot.
- `hash_object(data, obj_type, write=True)` computes the SHA-1 hash of the object and writes it to the object store.
- `write_file(path, data)` is used to write data bytes to files such as the reference files.
- `read_file(path)` reads file contents as bytes.
- The environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL` must be set for author information if not explicitly passed.

### Simplified Commit Creation Flow

```
+----------------------+
| Current Index State   |
+----------+-----------+
           |
           v
+----------------------+
| write_tree()         |  -- creates tree object from index
+----------+-----------+
           |
           v
+----------------------+
| get_local_master_hash()  -- fetches current master commit hash (parent)
+----------+-----------+
           |
           v
+----------------------+
| commit()             |  -- creates commit object referencing tree and parent
+----------+-----------+
           |
           v
+----------------------+
| Update refs/heads/master |
+----------------------+
```

---

## Summary

- The `commit` function encapsulates the current state of the working directory as recorded in the index into a commit object.
- The commit object includes metadata such as author, committer, timestamps, and parent commits.
- The local `master` branch reference is updated to point to the new commit.
- The current `master` commit hash can be retrieved using `get_local_master_hash`.
- Together, these functions form the core of pygit's commit management within the "Committing and Branching" section.