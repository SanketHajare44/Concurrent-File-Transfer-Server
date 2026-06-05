# concurrent-file-transfer-server

A process-based concurrent TCP file transfer server implemented in C on Linux. The server handles multiple simultaneous client connections by spawning an independent child process per connection using `fork()`, enabling parallel file downloads over a lightweight custom protocol.

---

## Table of Contents

- [Requirements](#requirements)
- [Building](#building)
- [Usage](#usage)
- [Protocol](#protocol)
- [Architecture](#architecture)
- [System Calls Reference](#system-calls-reference)
- [Error Handling](#error-handling)
- [Project Structure](#project-structure)
- [Limitations](#limitations)
- [Planned Improvements](#planned-improvements)
- [License](#license)

---

## Requirements

- Linux (kernel 4.x or later recommended)
- GCC 7.0 or later
- POSIX-compliant environment
- No external dependencies

---

## Building

Clone the repository and compile using GCC:

```sh
git clone https://github.com/your-username/concurrent-file-transfer-server.git
cd concurrent-file-transfer-server

gcc -Wall -Wextra -o server server.c
gcc -Wall -Wextra -o client client.c
```

> `-Wall -Wextra` flags are recommended to catch potential issues during compilation.

---

## Usage

### Starting the Server

```sh
./server <port>
```

**Example:**

```sh
./server 9000
```

```
Server is running on port: 9000
Waiting for client connections...
```

The server listens indefinitely until terminated with `Ctrl+C`.

---

### Running a Client

```sh
./client <server-ip> <port> <remote-filename> <local-filename>
```

**Example:**

```sh
./client 127.0.0.1 9000 Demo.txt Downloaded.txt
```

```
Header: OK 1024
File size: 1024 bytes
Transfer complete.
```

**Arguments:**

| Argument          | Description                        |
|-------------------|------------------------------------|
| `<server-ip>`     | IP address of the server           |
| `<port>`          | Port number the server is bound to |
| `<remote-filename>` | Name of the file to download     |
| `<local-filename>`  | Filename to save locally         |

---

### Testing Concurrent Connections

Launch multiple clients simultaneously to verify concurrent handling:

```sh
./client 127.0.0.1 9000 Demo.txt A.txt &
./client 127.0.0.1 9000 Demo.txt B.txt &
./client 127.0.0.1 9000 Demo.txt C.txt &
wait
```

Each client is served by an independent child process. All transfers proceed in parallel.

---

## Protocol

The server uses a simple custom application-level protocol over TCP.

### Request

The client sends a null-terminated or newline-terminated filename string:

```
Demo.txt\n
```

### Response

The server responds with a header before transmitting file data:

**On success:**
```
OK <file_size_in_bytes>\n
<raw file data>
```

**On failure:**
```
ERR\n
```

### Example Exchange

```
Client  →  "Demo.txt\n"
Server  →  "OK 2048\n"
Server  →  <2048 bytes of file data>
```

The client reads the header first to determine file size, then reads exactly that many bytes.

---

## Architecture

```
Server Process
│
├── socket() → bind() → listen()
│
└── Loop:
      accept()  ← blocks until client connects
         │
       fork()
         │
    ┌────┴────────────────────────────┐
    │                                 │
  Parent Process                Child Process
  (continues loop,              (handles this client:
   accepts next client)          reads filename,
                                 sends file,
                                 exits)
```

Each child process is fully isolated. The parent calls `wait()` or uses `SIGCHLD` to reap terminated children and avoid zombie processes.

---

## System Calls Reference

### Network Subsystem

| Call       | Purpose                              |
|------------|--------------------------------------|
| `socket()` | Create a TCP socket                  |
| `bind()`   | Bind socket to address and port      |
| `listen()` | Mark socket as passive (server)      |
| `accept()` | Accept an incoming client connection |
| `connect()`| Establish connection (client-side)   |
| `read()`   | Receive data over socket             |
| `write()`  | Send data over socket                |

### File Subsystem

| Call      | Purpose                              |
|-----------|--------------------------------------|
| `open()`  | Open requested file for reading      |
| `read()`  | Read file contents in chunks         |
| `write()` | Write file data to socket            |
| `stat()`  | Retrieve file size for protocol header |
| `close()` | Release file and socket descriptors  |

### Process Subsystem

| Call     | Purpose                                      |
|----------|----------------------------------------------|
| `fork()` | Spawn child process per client connection    |
| `wait()` | Reap terminated child processes              |
| `exit()` | Terminate child process after transfer       |

---

## Error Handling

The server handles the following failure cases:

| Condition              | Behavior                                      |
|------------------------|-----------------------------------------------|
| File not found         | Sends `ERR\n`, closes connection              |
| Invalid request format | Sends `ERR\n`, closes connection              |
| Client disconnects mid-transfer | Child process detects broken pipe and exits cleanly |
| `fork()` failure       | Logs error, closes client socket, continues  |
| Socket errors          | Logs `errno`, terminates affected child       |

---

## Project Structure

```
concurrent-file-transfer-server/
├── server.c          # Server: connection handling, process management, file transfer
├── client.c          # Client: connection, request, receive, save
├── sample_files/     # Test files for transfer verification
│   └── Demo.txt
├── README.md
└── LICENSE
```

---

<!-- ## Limitations

- File download only; upload not supported.
- No authentication or access control.
- No encryption; data is transmitted in plaintext.
- Large numbers of concurrent clients are limited by OS process limits (`ulimit`).
- No transfer resume or checksum verification.

---

## Planned Improvements

- [ ] File upload support
- [ ] Multithreaded variant using `pthread` for comparison
- [ ] SHA-256 checksum verification for transfer integrity
- [ ] SSL/TLS encryption using OpenSSL
- [ ] User authentication
- [ ] Transfer progress reporting on the client
- [ ] Structured server logging with timestamps

--- -->

## Author

**Sanket Sadashiv Hajare**

GitHub : [Link](https://github.com/SanketHajare44)  
LinkedIn : [Link](https://www.linkedin.com/in/sankethajare/)

If you found this project useful, please consider giving it a ⭐ on GitHub.


---

## License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.