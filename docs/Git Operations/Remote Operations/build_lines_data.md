# build_lines_data.md

## Overview

The `build_lines_data.py` module is responsible for constructing Git protocol packet lines used when sending commands to a Git server, particularly during push operations. This functionality is integral to the `pygit.push` process, where command lines need to be encoded in the specific Git packet line format before transmission. These packet lines ensure reliable communication with the remote Git server following the Git smart HTTP protocol.

Within the overall `pygit` documentation tree, this file resides under the **Remote Operations** section, which includes functions that handle communication and synchronization with remote repositories. It plays a crucial role in formatting the data payload sent to servers during remote updates, encapsulating commands such as reference updates and status reporting.

---

## Function Documentation

### `build_lines_data(lines)`

#### Purpose

Constructs a byte string formatted as Git protocol packet lines from a list of command lines. Each line is prefixed with its length in a 4-character hexadecimal format, followed by the line content and a newline character. A final flush packet (`0000`) is appended to indicate the end of the transmission.

This encoding complies with the Git protocol specification for packet lines, which is essential for correctly communicating with Git servers during operations like pushing commits.

#### Parameters

- `lines`: A list of byte strings, where each byte string represents a line of command or data to send to the Git server.

#### Returns

- A single `bytes` object containing all lines encoded as packet lines, including the final flush packet.

#### How it Works

1. Initialize an empty list to accumulate encoded packet lines.
2. For each given line:
   - Calculate the total length of the packet line as the length of the line plus 5 bytes (4 bytes for the length prefix and 1 for the newline).
   - Format the length as a 4-digit hexadecimal string.
   - Append the length prefix, the line itself, and a newline byte (`\n`) to the accumulator list.
3. After all lines are processed, append the flush packet `b'0000'` to signal the end.
4. Join and return the accumulated bytes as a single byte string.

#### Example Usage

```python
lines = [
    b'0000000000000000000000000000000000000000 1234567890abcdef1234567890abcdef12345678 refs/heads/master\x00 report-status',
    b'additional-command-data'
]

packet_data = build_lines_data(lines)

# packet_data now contains properly encoded Git packet lines ready for transmission.
```

---

## ASCII Diagram: Git Packet Line Structure

```
+--------------------+----------------------------+-----------+
| 4-byte Hex Length  |       Packet Line Data      |  Newline  |
|   (e.g., '001f')   |  (command or data bytes)    |   '\n'    |
+--------------------+----------------------------+-----------+
        |                        |                        |
        +------------------------+------------------------+
                             |
                         Repeated for each line
                             |
                          Final packet
                             |
                          +-------+
                          | '0000' |  <-- Flush packet indicating end of message
                          +-------+
```

This structure ensures the receiver knows exactly how many bytes to read for each packet line and when the transmission is complete.

---

## Related Functions in the Push Workflow

- **`pygit.push`**: Calls `build_lines_data` to encode the reference update commands before sending data to the remote server.
- **`pygit.create_pack`**: Creates the packfile data that follows the packet lines in the push request.
- **`pygit.http_request`**: Sends the encoded packet lines along with the packfile to the remote Git server.
- **`pygit.extract_lines`**: Parses the server’s response packet lines.

---

## Summary

The `build_lines_data` function is a fundamental utility for encoding commands in the Git packet line format. This encoding enables `pygit` to interact correctly with Git servers using the smart HTTP protocol, facilitating operations like pushing commits to remote repositories. Understanding this function is essential for anyone looking to extend or debug the remote communication aspects of `pygit`.