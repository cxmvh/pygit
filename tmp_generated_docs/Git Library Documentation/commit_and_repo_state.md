---
sidebar_position: 4
---

# Commit and Repository State

## Overview

This documentation file covers the processes and functions involved in managing the state of a Git repository with a focus on commits and repository structure. It explains how commit objects are created, branches updated, and how to retrieve and traverse the repository’s commit history. The file also details related functionality such as writing tree objects, hashing objects, and handling the Git index. This information is vital for understanding how the repository state is maintained and manipulated within the pygit library, supporting core workflows like committing changes, retrieving the current branch’s commit hash, and finding all objects reachable from commits and trees.

---

## commit

### Purpose

The `commit` function creates a new commit object in the repository representing the current state of the working directory as captured in the index. It updates the current branch to point to this new commit, thus advancing the repository history.

### Parameters

- `message` (string): The commit message describing the changes.
- `author` (string): The author of the commit.
- Additional optional parameters may include committer info and timestamp.

### Operation Steps

1. **Write Tree Object**  
   The function calls `write_tree` to construct a tree object representing the current index state, which captures the directory and file structure.

2. **Create Commit Object**  
   A commit object is created with the following attributes:  
   - Tree hash from the previous step.  
   - Parent commit hash, retrieved from the current branch.  
   - Author and committer information.  
   - Commit message.

3. **Hash the Commit Object**  
   The commit object is serialized and hashed using `hash_object` to produce its SHA-1 hash.

4. **Update Current Branch**  
   The branch reference is updated to point to the new commit hash, moving the branch forward.

5. **Return Commit Hash**  
   The function returns the SHA-1 hash of the newly created commit.

### Example

```python
commit_hash = commit(
    message="Fix bug in data processing",
    author="Jane Doe <jane@example.com>"
)
print(f"New commit created: {commit_hash}")
```

---

## get_local_master_hash

### Purpose

`get_local_master_hash` retrieves the current commit hash pointed to by the local master branch. This is useful for comparing repository states or determining the latest commit.

### Parameters

- None

### Operation Steps

1. Read the reference file for the master branch, typically `.git/refs/heads/master`.
2. Extract the commit hash stored in this reference.
3. Return the commit hash as a string.

### Example

```python
master_hash = get_local_master_hash()
print(f"Current master commit hash: {master_hash}")
```

---

## find_commit_objects

### Purpose

This function finds all commit objects reachable from a given commit hash. It traverses the commit history backward through parent commits to gather a complete set of commits.

### Parameters

- `start_commit_hash` (string): The SHA-1 hash of the commit to start traversal from.

### Operation Steps

1. Initialize a set or list to hold discovered commit hashes.
2. Begin with the `start_commit_hash` and add it to the set.
3. Read the commit object corresponding to the current hash.
4. Extract parent commit hashes from the commit object.
5. Recursively traverse each parent commit, adding unique hashes to the set.
6. Continue until commits with no parents are reached (root commits).
7. Return the set of all reachable commit hashes.

### Example

```python
commits = find_commit_objects(start_commit_hash="a1b2c3d4")
print("Reachable commits:")
for ch in commits:
    print(ch)
```

---

## find_tree_objects

### Purpose

`find_tree_objects` finds all tree objects reachable from a given tree hash. Trees represent directory structures; this function traverses the directory tree to collect all tree object hashes.

### Parameters

- `start_tree_hash` (string): The SHA-1 hash of the tree object to start traversal.

### Operation Steps

1. Initialize a collection to hold discovered tree hashes.
2. Add the `start_tree_hash` to this collection.
3. Read the tree object data corresponding to the hash.
4. For each entry in the tree:
    - If the entry is another tree (subdirectory), recursively traverse it.
    - If the entry is a blob (file), it is not further traversed.
5. Continue until all subtrees have been visited.
6. Return the set of all reachable tree hashes.

### Example

```python
trees = find_tree_objects(start_tree_hash="9f8e7d6c")
print("Reachable tree objects:")
for th in trees:
    print(th)
```

---

## Supporting Functions

### write_tree

Creates a tree object from the current index. It serializes directory contents and their modes into a tree object, which is then stored in the object database.

### hash_object

Hashes and stores a Git object (blob, tree, or commit) in the object database. It takes the object type and data, serializes it, computes the SHA-1 hash, and writes it to the `.git/objects` directory.

---

## ASCII Diagram: Commit Object Relationship

```
+-----------------+          +-----------------+          +-----------------+
| Commit C (HEAD) |  parent  | Commit B        |  parent  | Commit A (root) |
|  tree: T3       |  ----->  |  tree: T2       |  ----->  |  tree: T1       |
+-----------------+          +-----------------+          +-----------------+
        |                            |                            |
        v                            v                            v
   Points to                   Points to                   Points to
  Tree Object T3             Tree Object T2             Tree Object T1
```

This diagram illustrates a linear commit history where each commit points to a tree object representing the repository state and to its parent commit.

---

For additional details on object storage and repository initialization, refer to the related files in this documentation set:

- [Repository and Object Initialization](./repository_and_object_initialization.md)
- [Object Storage and Handling](./object_storage_and_handling.md)
- [Index and Working Copy](./index_and_working_copy.md)