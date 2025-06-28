# http_request.md

# Making Authenticated HTTP Requests to Git Servers

---

## Overview

The `http_request.md` file documents the function responsible for making authenticated HTTP requests to Git servers, a crucial part of remote operations in the pygit project. This functionality enables pygit to communicate securely with remote Git repositories, supporting commands such as pushing changes (`pygit.push`). Located under the **Remote Operations** section, this file complements other remote-related functions like fetching remote commit hashes and building Git protocol data packets.

The `http_request` function handles HTTP Basic Authentication, supports both GET and POST requests depending on whether data is provided, and returns raw response data from the server. It is a foundational utility used internally by higher-level operations to interact with remote Git servers.

---

## Function Documentation

### `http_request(url, username, password, data=None)`

#### Purpose

Make an authenticated HTTP request to a given URL with optional POST data. By default, performs a GET request if no data is provided, or a POST request if data bytes are supplied. This function supports HTTP Basic Authentication using the provided username and password.

#### Parameters

- `url` (str): The full URL to which the HTTP request is made (e.g., `https://github.com/user/repo.git/git-receive-pack`).
- `username` (str): Username for HTTP Basic Authentication.
- `password` (str): Password for HTTP Basic Authentication.
- `data` (bytes or None): Optional data to send in a POST request. If `None`, a GET request is made.

#### Returns

- `bytes`: The raw response data received from the server.

#### Operation Details

1. Creates a password manager and adds the provided credentials for the target URL.
2. Initializes an HTTP Basic Auth handler using the password manager.
3. Builds an opener with the auth handler to handle authentication automatically.
4. Opens the URL with the opener, sending `data` if provided (POST), otherwise as a GET request.
5. Reads and returns the raw bytes of the server's response.

This method abstracts away the details of HTTP request construction and authentication, making it straightforward to integrate with Git's HTTP protocol.

#### Example Usage

```python
from pygit import http_request

git_url = "https://github.com/exampleuser/example-repo.git/git-receive-pack"
username = "exampleuser"
password = "examplepass"

# Example GET request (fetch remote refs)
response_data = http_request(git_url.replace("/git-receive-pack", "/info/refs?service=git-receive-pack"), username, password)

# Example POST request (push data)
data_to_send = b"some git protocol data bytes"
push_response = http_request(git_url, username, password, data=data_to_send)

print("Response length:", len(push_response))
```

---

## Contextual Integration with pygit Push

The `http_request` function is an integral part of the `pygit.push` command flow, as illustrated below:

```ascii
+------------------+
| pygit.push       |
| (prepare push)   |
+---------+--------+
          |
          v
+------------------+
| get_remote_master_hash (uses http_request)  |
+------------------+
          |
          v
+------------------+
| create_pack (prepare pack data)             |
+------------------+
          |
          v
+------------------+
| http_request (POST push pack data)          |
+------------------+
          |
          v
+------------------+
| extract_lines (parse server response)       |
+------------------+
```

This flow shows how `http_request` enables communication with the remote Git server both for fetching refs and sending pack files during a push.

---

## Additional Notes

- The function uses `urllib.request` to handle HTTP operations and authentication.
- It supports both GET and POST methods transparently, depending on whether the `data` argument is provided.
- Proper error handling is expected to be implemented by callers (e.g., handling network errors or authentication failures).
- The returned raw response is typically processed by other pygit functions such as `extract_lines` to interpret Git protocol messages.

---

# Summary

This document covers the `http_request` function that performs authenticated HTTP communication with Git servers, supporting essential remote repository operations like pushes. It abstracts authentication and request handling, ensuring seamless integration into pygit's remote workflows.