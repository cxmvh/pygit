# http_request.md

# Making Authenticated HTTP Requests to Git Servers

## Overview

This document describes the `http_request` function used to perform authenticated HTTP requests to Git servers, a critical component within the `pygit` library's remote operations module. It facilitates secure communication with remote Git repositories, enabling commands such as pushing commits and querying remote references. The function supports both GET and POST requests with HTTP Basic Authentication, which is typically required when interacting with Git servers over HTTP(S).

Within the broader documentation tree, this file belongs to the **Remote Operations** section, a collection of functions that manage communication and synchronization with remote repositories. It is used primarily by the `push` command implementation (`pygit.push`), which pushes local commits to a remote Git repository by sending packed objects and Git protocol commands via HTTP.

---

## Function Documentation

### `http_request(url, username, password, data=None)`

#### Purpose

The `http_request` function makes an authenticated HTTP request to a given URL with optional data payload. It handles HTTP Basic Authentication using the provided username and password. When `data` is supplied, it sends a POST request; otherwise, it defaults to a GET request.

This function is essential for communicating with Git servers over HTTP(S), especially for operations like pushing objects to a remote repo or retrieving remote references.

#### Parameters

- `url` (`str`): The full URL of the Git server endpoint to send the request to. Examples include:
  - `<git_url>/info/refs?service=git-receive-pack` for querying remote refs
  - `<git_url>/git-receive-pack` for pushing objects

- `username` (`str`): Username used for HTTP Basic Authentication.

- `password` (`str`): Password used for HTTP Basic Authentication.

- `data` (`bytes`, optional): Optional bytes payload to send in a POST request. If `None`, a GET request is performed.

#### Returns

- `bytes`: The raw response data read from the HTTP response.

#### Behavior

1. Creates a `HTTPPasswordMgrWithDefaultRealm` and adds the given username and password for the URL.

2. Builds an `HTTPBasicAuthHandler` with the password manager to handle authentication automatically.

3. Constructs an opener using the authentication handler.

4. Opens the URL with the optional `data` payload:
    - If `data` is `None`, a GET request is made.
    - If `data` is provided, a POST request is made with the payload.

5. Reads and returns all response bytes from the server.

#### Usage Example

```python
from pygit import http_request

git_url = "https://example.com/myrepo.git"
username = "alice"
password = "s3cr3t"

# Example 1: GET request to fetch remote refs
info_refs_url = f"{git_url}/info/refs?service=git-receive-pack"
response_data = http_request(info_refs_url, username, password)
print("Received info/refs data:", response_data[:100], "...")

# Example 2: POST request to push data
push_url = f"{git_url}/git-receive-pack"
push_data = b"...binary pack data..."
response = http_request(push_url, username, password, data=push_data)
print("Push response:", response)
```

---

## Additional Context: How `http_request` Fits in the Push Workflow

The `http_request` function is called internally by the `push` function to communicate with the remote Git server during the push process. Below is a simplified ASCII diagram illustrating this interaction in the push flow:

```
+-----------------+                       +------------------+
|   Local pygit   |                       |   Remote Git     |
|    Client       |                       |    Server        |
+-----------------+                       +------------------+
        |                                            |
        | 1. GET /info/refs?service=git-receive-pack |
        |------------------------------------------->|
        |                                            |
        |              2. Receive refs data          |
        |<-------------------------------------------|
        |                                            |
        | 3. Prepare pack data and commands          |
        |                                            |
        | 4. POST /git-receive-pack with pack data   |
        |------------------------------------------->|
        |                                            |
        |          5. Receive push status lines       |
        |<-------------------------------------------|
        |                                            |
```

In step 1 and 4, the `http_request` function is used to send HTTP requests with authentication.

---

## Summary

The `http_request` function abstracts the complexity of making authenticated HTTP requests to Git servers, supporting both GET and POST methods with HTTP Basic Authentication. It is a foundational utility enabling remote synchronization operations such as pushing commits and fetching remote references in the `pygit` toolset.