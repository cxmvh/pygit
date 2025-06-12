# HTTP Request Handling and Authentication

## Overview

The `http_request.md` file documents the HTTP request handling and authentication mechanisms used within the `pygit` project. This module is a critical part of the internal mechanics supporting Git operations that communicate with remote repositories, particularly for the `push` command. It enables authenticated HTTP communication with Git servers, handling both GET and POST requests as needed during repository synchronization. The functions detailed here facilitate requesting remote repository references, sending packfiles, and managing authentication credentials robustly and securely.

Within the broader project documentation tree, this file resides under *Internal Mechanics*, highlighting its role as an essential utility leveraged by high-level commands (`pygit.push`) to interact with remote Git servers.

---

## Function Documentation

### `http_request(url, username, password, data=None)`

**Purpose:**  
Performs an HTTP request to the specified URL using Basic Authentication. It defaults to a GET request unless `data` is provided, in which case it sends a POST request with the given data payload.

**Parameters:**  
- `url` (str): Target URL for the HTTP request.  
- `username` (str): Username for HTTP Basic Authentication.  
- `password` (str): Password for HTTP Basic Authentication.  
- `data` (bytes or None): Data to send in a POST request. If `None`, a GET request is made.

**Operation Detail:**  
1. Creates a password manager and adds credentials for the target URL.  
2. Builds an HTTP Basic Auth handler with the password manager.  
3. Constructs an opener using the auth handler.  
4. Opens the URL, using POST if `data` is provided.  
5. Reads and returns the response bytes.

**Usage Example:**

```python
response_bytes = http_request(
    url='https://example.com/git-receive-pack',
    username='gituser',
    password='securepass',
    data=b'packfile data bytes'
)
```

---

### `get_remote_master_hash(git_url, username, password)`

**Purpose:**  
Fetches the current commit hash (SHA-1 hex string) of the remote repository's `master` branch, or returns `None` if the branch has no commits.

**Parameters:**  
- `git_url` (str): Base URL of the remote Git repository.  
- `username` (str): Username for authentication.  
- `password` (str): Password for authentication.

**Operation Detail:**  
1. Constructs the URL to request refs info for the `git-receive-pack` service.  
2. Calls `http_request` to fetch refs advertisement from the remote.  
3. Extracts lines from the response using `extract_lines`.  
4. Validates the service header and empty line.  
5. Parses the line containing the master branch hash and ref name.  
6. Returns the SHA-1 hash as a string or `None` if no commits exist.

**Usage Example:**

```python
remote_hash = get_remote_master_hash(
    git_url='https://example.com/myrepo.git',
    username='gituser',
    password='securepass'
)
print(f"Remote master hash: {remote_hash or 'No commits'}")
```

---

### `extract_lines(data)`

**Purpose:**  
Parses the Git smart HTTP protocol packet lines from raw server response bytes into a list of byte strings.

**Parameters:**  
- `data` (bytes): Raw byte response from Git server.

**Operation Detail:**  
- Iteratively reads 4-byte hexadecimal length prefixes.  
- Extracts each packet line based on its length.  
- Stops when a zero-length packet (`0000`) is encountered or data ends.

**Usage Example:**

```python
lines = extract_lines(server_response_bytes)
for line in lines:
    print(line)
```

---

### `build_lines_data(lines)`

**Purpose:**  
Encodes a list of byte strings into Git smart HTTP packet lines format for transmission.

**Parameters:**  
- `lines` (list of bytes): Each item is a line to encode.

**Operation Detail:**  
- For each line, prefix with 4-byte hex length = line length + 5 (for length and newline).  
- Append the line and a newline byte.  
- Ends with a flush packet `0000`.

**Usage Example:**

```python
lines = [b'abc 123 refs/heads/master\x00 report-status']
packet_data = build_lines_data(lines)
```

---

### `push(git_url, username=None, password=None)`

**Purpose:**  
Pushes the local `master` branch to the specified remote Git repository URL.

**Parameters:**  
- `git_url` (str): URL of the remote Git repository.  
- `username` (str or None): Username for authentication. Defaults to environment variable `GIT_USERNAME` if None.  
- `password` (str or None): Password for authentication. Defaults to environment variable `GIT_PASSWORD` if None.

**Operation Detail:**  
1. Retrieves remote and local `master` commit hashes.  
2. Finds missing objects that exist locally but not remotely.  
3. Prints status about updating remote master.  
4. Builds Git protocol lines announcing old and new refs.  
5. Creates a packfile of the missing objects.  
6. Sends packfile to remote via authenticated HTTP POST request.  
7. Extracts and validates response lines to confirm success.  
8. Returns the remote SHA-1 and set of missing objects.

**Usage Example:**

```python
remote_sha1, missing_objects = push(
    git_url='https://example.com/myrepo.git',
    username='gituser',
    password='securepass'
)
print(f"Pushed to remote {remote_sha1}, missing objects sent: {len(missing_objects)}")
```

---

## ASCII Diagram: Git Push HTTP Request Flow

```
Local Repository                        Remote Git Server
----------------                        -----------------
   |                                           |
   |-- get_remote_master_hash() -------------->|
   |                                           |  (GET refs info)
   |<------------------------------------------|
   |                                           |
   |-- find_missing_objects()                   |
   |                                           |
   |-- build_lines_data()                       |
   |                                           |
   |-- create_pack(missing_objects)             |
   |                                           |
   |-- http_request(url='/git-receive-pack',   |
   |                 data=packfile_data) ----->|
   |                                           |  (POST packfile)
   |<------------------------------------------|
   |                                           |
   |-- extract_lines(response)                  |
   |                                           |
   |-- Confirm 'unpack ok' and 'ok refs/heads/master' lines
   |                                           |
   `--> Push successful!                        |
```

---

## Summary

This module abstracts the lower-level details of the Git smart HTTP protocol, providing authenticated request handling and data packet management that enables pushing commits and objects to remote repositories. Functions like `http_request`, `get_remote_master_hash`, and `push` integrate tightly to authenticate, negotiate, and transfer data securely. Understanding these functions is key to extending or debugging the `pygit` push command’s network communication.