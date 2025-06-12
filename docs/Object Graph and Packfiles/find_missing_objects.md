# find_missing_objects.md

---

## Overview

This document describes the functionality provided by the `find_missing_objects` module, which is responsible for determining which Git objects are present locally but missing on a remote repository. This is a crucial step in Git operations such as pushing commits, where it is necessary to identify and transfer only those objects that the remote repository does not yet have. The module fits within the broader "Object Graph and Packfiles" section of the documentation tree, which focuses on managing commit graphs, object sets, and packfile creation.

The ability to efficiently find missing objects ensures minimal data transfer during synchronization operations and underpins the correctness of distributed version control workflows. This document also illustrates how `find_missing_objects` interacts with other core functions like `find_commit_objects` and how it integrates into the push operation.

---

## Function Documentation

### `find_missing_objects(local_sha1, remote_sha1)`

Determines the set of Git object hashes that exist in the local commit history but are missing from the remote commit history.

#### Purpose

When pushing changes to a remote repository, Git needs to know which objects (commits, trees, blobs) the remote repository already has and which it lacks. `find_missing_objects` compares the local and remote commit graphs to find all objects that must be sent to the remote.

#### Parameters

- `local_sha1` (str): The SHA-1 hash of the local commit (usually the tip of the local branch, e.g., `master`).
- `remote_sha1` (str or None): The SHA-1 hash of the remote commit (tip of the remote branch). If `None`, it indicates the remote has no commits.

#### Returns

- `set[str]`: A set of SHA-1 hash strings representing objects present locally but missing remotely.

#### Preconditions

- Both `local_sha1` and `remote_sha1` should be valid commit hashes or `remote_sha1` can be `None`.
- The commits must be reachable in the local and remote repositories respectively.

#### How it works

1. Calls `find_commit_objects(local_sha1)` to collect all objects reachable from the local commit, including the commit itself, its tree, and parent commits.
2. If `remote_sha1` is `None`, implying the remote repo is empty, returns all local objects.
3. Otherwise, calls `find_commit_objects(remote_sha1)` to get objects reachable from the remote commit.
4. Returns the set difference: all local objects minus all remote objects, resulting in the objects missing remotely.

#### Example usage

```python
local_commit = "a1b2c3d4e5f678901234567890abcdef12345678"
remote_commit = "9f8e7d6c5b4a3210fedcba9876543210abcdef12"

missing_objects = find_missing_objects(local_commit, remote_commit)
print(f"Objects missing remotely: {len(missing_objects)}")
for obj in sorted(missing_objects):
    print(obj)
```

---

## Related Function: `find_commit_objects(commit_sha1)`

To understand `find_missing_objects`, it is important to grasp how `find_commit_objects` works, as it gathers all the objects reachable from a commit.

#### Purpose

Recursively collects all Git objects (commits, trees, blobs) reachable from a given commit SHA-1.

#### Key steps

- Reads the commit object.
- Parses the commit to find its tree and parent commits.
- Recursively collects objects from the tree and all parents.
- Returns a set of SHA-1 hashes including the commit itself.

#### ASCII Diagram: Commit Graph Traversal

```
          +------------------+
          | Commit SHA-1     |
          +------------------+
                   |
                   v
          +------------------+
          | Tree SHA-1        |
          +------------------+
           /         \
          /           \
+-----------+    +-----------+
| Blob SHA1 |    | Blob SHA1 |
+-----------+    +-----------+

Commit may have Parent Commit(s) --> Recursively traverse parents similarly.
```

---

## Integration in Push Workflow

Here is a simplified diagram illustrating how `find_missing_objects` fits into a typical Git push operation:

```
Local Repository                     Remote Repository
+----------------+                 +----------------+
| Local Commit   |                 | Remote Commit  |
| SHA: local_sha1|                 | SHA: remote_sha1|
+--------+-------+                 +--------+-------+
         |                                  |
         | find_missing_objects(local_sha1, remote_sha1)
         |--------------------------------->|
         |                                  
         |   Returns set of missing objects
         |<---------------------------------
         |
    create_pack(missing_objects)
         |
         |  Send packfile containing missing objects
         |----------------------------------->
         |
         |  Remote updates its refs/heads/master
         |<-----------------------------------
```

---

# Summary

The `find_missing_objects` function is a core utility that enables efficient synchronization between local and remote Git repositories by identifying the minimal set of objects that need to be transferred. It leverages commit and tree traversal to build sets of reachable objects and performs set subtraction to find missing objects.

This method ensures that Git push operations only transmit new data, conserving bandwidth and maintaining the integrity of the distributed version control system.