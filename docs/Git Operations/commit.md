# commit.md

# How to Create Commits and Update the Master Branch

---

## Overview

This document explains how to create commits and manage updates to the `master` branch within the pygit repository system. It forms part of the **Committing and Branching** section in the broader pygit documentation tree, which covers processes related to committing changes and branch management.

The commit process in pygit involves capturing the current state of the repository's index (staged files), creating a commit object that references this snapshot, and updating the local reference to the `master` branch. Additionally, updating the `master` branch may include pushing commits to a remote repository.

Key functions covered here include:

- `commit()` — Create a new commit from the current index.
- `get_local_master_hash()` — Retrieve the current commit hash on the local `master` branch.
- `push()` — Push local commits on `master` to a remote repository.

This file also references related functions for building tree objects, managing object storage, and interacting with remote repositories.

---

## Function Documentation

### `commit(message, author=None)`

**Purpose:**  
Create a new commit object from the current index state, update the local `master` branch reference to this commit, and return the commit hash.

**Parameters:**  
- `message` (str): The commit message describing the changes.  
- `author` (str, optional): The commit author's info in the format `"Name <email>"`. Defaults to environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Operation Steps:**  
1. Calls `write_tree()` to write a tree object representing the current index state.  
2. Reads the current commit hash of the local `master` branch using `get_local_master_hash()` to set as a parent commit (if any).  
3. Constructs author and committer metadata including timestamp and timezone offset.  
4. Assembles commit object content with references to the tree, parent commit (if any), author, committer, and commit message.  
5. Hashes and stores the commit object using `hash_object()` with type `"commit"`.  
6. Updates the `.git/refs/heads/master` file to reference the new commit hash.  
7. Prints confirmation of the new commit hash.

**Example Usage:**
```python
commit_hash = commit("Fix issue with file parsing")
print(f"Created commit: {commit_hash}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the SHA-1 commit hash string for the current local `master` branch, or `None` if no commits exist yet.

**Parameters:**  
None

**Operation Steps:**  
1. Reads the contents of the `.git/refs/heads/master` file.  
2. Decodes and strips whitespace from the read commit hash string.  
3. Handles the case where the file does not exist by returning `None`.

**Example Usage:**
```python
current_master = get_local_master_hash()
if current_master:
    print(f"Current master commit: {current_master}")
else:
    print("No commits found on master branch.")
```

---

### `push(git_url, username=None, password=None)`

**Purpose:**  
Push the local `master` branch commits to a remote Git repository URL.

**Parameters:**  
- `git_url` (str): The remote Git repository URL.  
- `username` (str, optional): Username for HTTP authentication; defaults to environment variable `GIT_USERNAME`.  
- `password` (str, optional): Password for HTTP authentication; defaults to environment variable `GIT_PASSWORD`.

**Operation Steps:**  
1. Retrieve the SHA-1 hash of the remote `master` branch using `get_remote_master_hash()`.  
2. Retrieve the local `master` branch hash using `get_local_master_hash()`.  
3. Determine which objects are present locally but missing remotely with `find_missing_objects()`.  
4. Print an update message showing the remote and local commit hashes and number of objects to push.  
5. Construct Git protocol packet lines to announce the ref update.  
6. Create a pack file containing the missing objects using `create_pack()`.  
7. Send an authenticated HTTP POST request to the remote `/git-receive-pack` endpoint with the update and pack file data.  
8. Extract and validate server response lines to confirm success.  
9. Return a tuple of the previous remote SHA-1 and the set of missing objects pushed.

**Example Usage:**
```python
remote_url = "https://example.com/myrepo.git"
push(remote_url)
```

---

## Additional Supporting Functions

- **`write_tree()`**: Generates a tree object based on the current index entries, capturing the snapshot of files staged for commit.

- **`hash_object(data, obj_type, write=True)`**: Hashes and optionally writes an object (blob, tree, commit) to the object store.

- **`write_file(path, data)`**: Writes binary data to a file, used for updating refs or object files.

- **`get_remote_master_hash(git_url, username, password)`**: Queries the remote repository to obtain the current commit hash of the remote `master` branch.

- **`find_missing_objects(local_sha1, remote_sha1)`**: Compares the local and remote commits to determine which objects need to be pushed.

- **`create_pack(objects)`**: Creates a pack file containing all Git objects to transfer efficiently.

---

## ASCII Diagram: Commit Creation and Push Flow

```
Working Directory
       |
       v
   [Add files]
       |
       v
   Index (Staging Area)
       |
       v
  write_tree() -> Tree Object (snapshot)
       |
       v
commit() creates Commit Object referencing Tree and optionally Parent Commit
       |
       v
Update local ref: .git/refs/heads/master
       |
       v
push() ----> Query remote master ref
       |            |
       |      if remote lacks commits
       |            |
       |      find missing objects locally
       |            |
       |      create pack file with missing objects
       |            |
       +--------> Send pack file + update info to remote
                    |
                    v
             Remote receives and applies updates
```

---

## Summary

This document outlines the core procedures for creating commits and updating the master branch in pygit. The `commit()` function captures the current staged state into a commit object and updates the local reference. The `push()` function facilitates synchronizing these commits with a remote repository, ensuring that all necessary objects are transferred efficiently.

For a complete understanding, see related documentation on index management, object handling, and remote operations within the pygit project.