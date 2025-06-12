# status_and_diff.md

## Overview

This document provides detailed descriptions of functions responsible for status reporting and diff display within the repository. These functions form a critical part of the **Status and Diff** section of the project, enabling users and tools to determine the current state of the working directory relative to the index and repository, and to visualize differences between file versions. This file complements other documentation in the repository dealing with status detection (`status.md`), diff generation (`diff.md`), and index management (`add.md`, `get_status.md`), by focusing on the specific implementations of status reporting and diff display logic.

---

## Function Documentation

### `report_status`

**Purpose:**  
Generates a comprehensive status report of the working directory by comparing the current working copy files, the Git index, and the last committed tree.

**Parameters:**  
- `repo`: The repository object representing the current Git repository.  
- `index`: The Git index entries representing the stage area.  
- `working_dir_files`: A list or mapping of files present in the working directory along with their metadata.

**Operation:**  
1. Scan the index and working directory to identify file changes, including:  
   - New files (untracked files in working directory).  
   - Modified files (files differing between index and working directory).  
   - Deleted files (files removed from working directory but present in index).  
2. Classify files into status categories such as staged, unstaged, untracked, or conflicted.  
3. Aggregate the status information into a structured report that can be used for user display or further processing.

**Example Usage:**

```python
repo = open_repository('/path/to/repo')
index = read_git_index(repo)
working_files = scan_working_directory(repo.worktree)

status_report = report_status(repo, index, working_files)
print_status_report(status_report)
```

---

### `display_diff`

**Purpose:**  
Displays the differences between two versions of files, typically between the index and the working directory, or between commits and the index.

**Parameters:**  
- `file_path`: The path to the file to diff.  
- `old_version`: Content or object representing the old file version (e.g., blob from index or commit).  
- `new_version`: Content or object representing the new file version (e.g., working directory file content).  
- `context_lines`: Number of context lines to show around changes (optional, default typically 3).

**Operation:**  
1. Read and decode the old and new file contents.  
2. Compute the line-by-line differences using a diff algorithm (e.g., Myers diff).  
3. Format the output with standard diff markers (`---`, `+++`, `@@`) and line prefixes (`+`, `-`, ` `).  
4. Respect the `context_lines` parameter to limit output to relevant changes.  
5. Render the diff output to the console or a specified output stream.

**Example Usage:**

```python
old_blob = get_blob_from_index(repo, 'README.md')
new_content = read_working_file(repo.worktree, 'README.md')

display_diff('README.md', old_blob.data, new_content)
```

---

## ASCII Diagram: Status Reporting Flow

```
+----------------+       +------------+       +-----------------+
| Working Dir    |       | Git Index  |       | Last Commit Tree |
| (Current files)|       | (Stage)    |       | (HEAD commit)    |
+-------+--------+       +-----+------+       +---------+-------+
        |                      |                        |
        +------ Compare -------+--------- Compare ------+
               Changes                  Changes
                  |                          |
                  +------------+-------------+
                               |
                        report_status()
                               |
                       +-------+--------+
                       | Status Report  |
                       +----------------+
```

---

## ASCII Diagram: Diff Display Example

```
--- a/README.md
+++ b/README.md
@@ -10,7 +10,8 @@
-This is the old line in the file.
+This is the new line in the file.
+An additional new line added here.
```

---

This document serves as a reference for developers and users looking to understand or extend the status and diff functionalities of the repository. For more context, see related files like `status.md` for status detection algorithms, and `diff.md` for detailed diff command explanations.