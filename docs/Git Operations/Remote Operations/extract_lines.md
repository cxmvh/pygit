# extract_lines.md

# Extracting Lines from Git Protocol Server Responses

## Overview

This documentation covers the `extract_lines` function, which is essential for parsing raw data received from Git protocol server responses. When interacting with a Git remote repository — particularly during operations like pushing (`pygit.push`) — the client communicates with the server using the Git Smart HTTP protocol. This protocol transmits data in a packet line format, where each packet line is prefixed with a 4-byte hexadecimal length field, followed by the payload.

The `extract_lines` function takes the raw byte stream returned by the server and extracts the individual message lines by decoding these packet lines. This function is a critical component in the remote communication stack, enabling the client to interpret server responses such as status messages, refs updates, and error reports.

In the pygit documentation tree, this file resides under the "Remote Operations" section, illustrating its role in managing communication with remote repositories. It is closely related to other functions like `http_request`, `build_lines_data`, and `get_remote_master_hash` that together handle the Git push process.

---

## Function: `extract_lines(data)`

### Purpose

Extract a list of individual lines from the raw byte stream returned by a Git protocol server. The input `data` is expected to be a sequence of packet lines encoded according to the Git Smart HTTP protocol.

Each packet line consists of:

- A 4-byte ASCII hexadecimal length prefix (including the 4 length bytes themselves).
- The payload of the indicated length minus 4 bytes.
- A length of zero (`0000`) indicates the end of the packet stream.

This function decodes these packet lines, returning a list of raw payload lines (bytes), excluding the length prefixes.

### Parameters

- `data` (`bytes`): The raw byte data returned from the server, containing one or more Git protocol packet lines.

### Returns

- `List[bytes]`: A list of byte strings, each corresponding to one decoded payload line from the server response.

### Preconditions

- The input `data` must conform to the Git packet line protocol format.
- The data should not contain more than 1000 packet lines (a safety limit to avoid infinite loops).

### Operation Details

1. Initialize an empty list `lines` to store the extracted lines.
2. Set an index pointer `i` to 0 to track the current position in the byte stream.
3. Loop up to 1000 times (to prevent runaway loops):
   - Read 4 bytes from `data[i:i+4]` and decode them as a hexadecimal integer to get `line_length`.
   - Extract the payload from `data[i+4:i+line_length]`.
   - Append the payload line to `lines`.
   - If `line_length` is zero (`0000`), this signals the end of the packet stream; advance `i` by 4 bytes.
   - Otherwise, advance `i` by `line_length` bytes to move to the next packet line.
   - Break the loop if `i` reaches or exceeds the length of `data`.
4. Return the list of extracted lines.

### Example Usage

Suppose you performed an HTTP request to a Git receive-pack service and received a raw response `response_data`:

```python
response_data = b'0012# service=git-receive-pack\n0000'  # simplified example response
lines = extract_lines(response_data)
for idx, line in enumerate(lines):
    print(f"Line {idx + 1}: {line}")
```

Output:

```
Line 1: b'# service=git-receive-pack\n'
Line 2: b''
```

In a real push operation, the lines may include status messages such as:

```
b'unpack ok\n'
b'ok refs/heads/master\n'
```

### Integration in Push Flow

Within the push process (`pygit.push`), after sending the pack data to the remote, the client receives a response containing packet lines indicating the result of the unpack and update operations on the server. The `extract_lines` function is used to parse these lines for validation and reporting.

---

## ASCII Diagram: Packet Line Format

```
+-----------------+-----------------------+
| 4-byte length   |       Payload          |
| (hex ASCII)     | (line_length - 4 bytes)|
+-----------------+-----------------------+

Example packet line (hex view):

  30 30 31 32 23 20 73 65 72 76 69 63 65 3d 67 69 74 2d 72 65 63 65 69 76 65 2d 70 61 63 6b 0a
  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
  0  0  1  2  #  ' ' s  e  r  v  i  c  e  =  g  i  t  -  r  e  c  e  i  v  e  -  p  a  c  k \n

Length: '0012' hex = 18 decimal bytes total
Payload: '# service=git-receive-pack\n' (14 bytes)
```

---

# Summary

The `extract_lines` function is a fundamental utility for interpreting Git Smart HTTP protocol responses by parsing the variable-length, length-prefixed packet lines into manageable byte lines. It is an indispensable part of the remote push operation in pygit, enabling reliable communication and response validation with Git servers.