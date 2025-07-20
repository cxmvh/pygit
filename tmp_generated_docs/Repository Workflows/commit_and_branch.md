---
sidebar_position: 1
---

# Creating Commits and Managing Branch References

## Overview

This document details the processes involved in creating commit objects, updating branch references, and managing commit parent relationships within a Git repository. It covers the mechanisms for writing tree objects that represent snapshots of the working directory, hashing objects for integrity, and updating branch heads to point to new commits. This file fits within the broader "Repository Workflows" section, providing developers with a clear understanding of how commits and branches are manipulated at a low level in the repository.

---

## Creating a Commit Object

### Purpose

The commit creation function is responsible for generating a new commit object that records the state of the repository at a certain point in time. This includes linking to a tree object that captures the directory snapshot, referencing parent commits, author and committer information, and the commit message.

### Parameters

- **tree_hash**: The SHA-1 hash of the tree object representing the working directory snapshot.
- **parent_hashes**: A list of zero or more SHA-1 hashes of parent commits (empty for initial commits).
- **author**: Metadata about the author (name, email, timestamp).
- **committer**: Metadata about the committer (often same as author).
- **message**: The commit message describing the changes.

### Operation Steps

1. **Prepare commit content**: Assemble the commit data in the standard Git format, including:
    - The tree hash line.
    - One or more parent commit lines (if applicable).
    - Author and committer lines with timestamps.
    - A blank line followed by the commit message.

2. **Hash and store the commit**:
    - Compute the SHA-1 hash of the commit content, prefixed with the "commit" type and length.
    - Store the resulting object in the Git object database under its hash.

3. **Return the commit hash**: This hash uniquely identifies the new commit object.

### Example Usage

```python
commit_hash = create_commit(
    tree_hash="a1b2c3d4e5f67890123456789abcdef123456789",
    parent_hashes=["123456789abcdef123456789abcdef1234567890"],
    author="John Doe <john@example.com> 1686500000 +0200",
    committer="John Doe <john@example.com> 1686500000 +0200",
    message="Fix bug in data processing module"
)
print(f"Created commit {commit_hash}")
```

---

## Writing Tree Objects

### Purpose

Tree objects represent directories and contain references to blobs (files) and subtrees (subdirectories). The function to write trees converts the current state of the index or working directory into a tree object that can be referenced by a commit.

### Parameters

- **entries**: A list of tuples representing files and subdirectories. Each tuple contains:
  - Mode (e.g., `100644` for files, `40000` for directories)
  - Filename
  - SHA-1 hash of the blob or subtree object

### Operation Steps

1. **Format each entry**: Concatenate the mode, filename, and SHA-1 hash in Git's tree object format.
2. **Concatenate all entries**: Form the complete tree content.
3. **Hash and store**: Compute the SHA-1 hash with the "tree" prefix and store the object.
4. **Return the tree hash**: This hash represents the snapshot of the directory.

### Example Usage

```python
tree_hash = write_tree([
    (0o100644, "README.md", "d670460b4b4aece5915caf5c68d12f560a9fe3e4"),
    (0o40000, "src", "4b825dc642cb6eb9a060e54bf8d69288fbee4904")
])
print(f"Written tree object {tree_hash}")
```

---

## Updating Branch References

### Purpose

When a new commit is created, the branch reference (e.g., `refs/heads/main`) must be updated to point to this commit hash. This function safely updates the branch head to maintain repository consistency.

### Parameters

- **branch_name**: The name of the branch to update (e.g., `"main"`).
- **commit_hash**: The SHA-1 hash of the new commit to set as the branch head.

### Operation Steps

1. **Locate the branch reference file**: This typically resides in `.git/refs/heads/{branch_name}`.
2. **Write the new commit hash**: Overwrite the reference file with the new commit hash followed by a newline.
3. **Ensure atomic update**: Use atomic write operations to prevent corruption.
4. **Optionally update the HEAD**: If the branch is currently checked out, update HEAD to point to the new commit.

### Example Usage

```python
update_branch("main", "a1b2c3d4e5f67890123456789abcdef123456789")
print("Branch 'main' updated to new commit.")
```

---

## Managing Commit Parents

### Purpose

Commit parent management ensures the correct ancestry of commits, enabling Git to track history and merge branches. This includes handling initial commits with no parents and merge commits with multiple parents.

### Key Points

- Initial commits have no parents.
- Regular commits have one parent.
- Merge commits have two or more parents.
- Parent hashes are included in commit objects to maintain history linkage.

### ASCII Diagram: Commit Parent Relationships

```
Initial commit (no parents)
       |
   Commit A
       |
   Commit B
      /  \
 Commit C  Commit D (merge commit with parents C and D)
```

This diagram shows how commits relate to each other, with merge commits having multiple parents.

---

## Hashing Objects

### Overview

Git uses SHA-1 hashes to uniquely identify objects such as commits, trees, and blobs. The hashing process includes:

- Prepending the object type and size to the content.
- Computing the SHA-1 hash over this data.
- Storing the object under its hash in the `.git/objects` directory.

### Example Operation

For a commit object:

```
"commit {content_length}\0{commit_content}"
```

where `{commit_content}` includes the tree, parent(s), author, committer, and message.

---

By understanding these components—commit creation, tree writing, branch updating, and parent management—developers can grasp how Git internally manages repository history and state transitions. This knowledge facilitates advanced operations such as scripting custom commits or troubleshooting repository inconsistencies.