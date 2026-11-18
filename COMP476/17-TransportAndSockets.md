# COMP476 — Networked Computer Systems
## Lecture 17 — Transport Layer and Sockets (slides: "Transport Layer and Sockets," Mon Oct 19, 2026; reading 5.2 – 5.4)

**What's new vs. Lecture 16:** slides 1–31 are a **condensed re-run of the Transport layer** (ports, UDP, TCP, handshake, flow/congestion control, reliability). The **new material** is the **socket interface**: socket methods and program flow, Python and Java TCP client/server code, UDP sockets, reading a URL, finding your own address, and how web servers use threads.

> Full detail on TCP/UDP internals is in `Lecture-16-TransportLayer-Notes.md`. §1 below is the quick recap; §2 onward is new.

---

## 1. Transport layer recap (from Lecture 16)

- Transport is the **first end-to-end layer**: only the two end hosts run it.
- **Port** (16 bits) → which **application** on the host. IP → which **host**.

**Well-known ports ★**
| Port | Service | Port | Service |
|---|---|---|---|
| 20, 21 | FTP | 80 | HTTP |
| 22 | SSH | 110 | POP3 |
| 23 | Telnet | 123 | NTP |
| 25 | SMTP | 143 | IMAP |
| 53 | DNS | 443 | HTTPS |

**UDP:** sends individual datagrams; may be **lost, delayed, corrupted, out of order**. Header has **port numbers + checksum** (header and data). Checksum **optional in IPv4, mandatory in IPv6**.

**TCP:** **connection-oriented, reliable byte stream.** IP sends packets; TCP **merges the data into a byte stream** and fixes lost / corrupted / out-of-order / delayed packets.

| Mechanism | Key fact |
|---|---|
| **Handshake** ★ | Client **SYN** → Server **SYN+ACK** → Client **ACK**. Sequence numbers are established. |
| **Closing** | Each direction closes **independently** with a **FIN** (closed when ACKed). Connection closed when **both** directions are done. Usually the other side replies **ACK+FIN**. |
| **Listening** | Server **listens** on a port. Client's OS assigns a **temporary port**. |
| **Flow control** ★ | **Sliding window**: receiver says how many bytes it can take; sender can't exceed it. (Slide: send 2K, told 2K available → send 2K more → window full → wait.) |
| **Sequence numbers** | Count **bytes sent**. Duplicate seq number → **discard**. ACKs carry a seq number back. |
| **Initial seq number** ★ | Sent in the SYN-ACK handshake. **Random**, not always 1 → defends against **TCP sequence prediction attacks**. |
| **Timeout retransmit** | Timer expires → assume lost → resend with **original seq number**; next timer is **2×**. |
| **Timeout too short** | Unnecessary retransmits; sender may mistake the original's ACK for the retransmission's. |
| **Stop-and-wait** | Low throughput → TCP sends **multiple packets before waiting for an ACK**. |
| **Duplicate ACK** ★ | **3 duplicate ACKs** with the same number → retransmit the last unacknowledged packet. |
| **Cumulative ACK** ★ | ACK covers everything **up to** that number. Lost #100 → receiver must **re-ACK 99**. Got 6 and 7 but missed 5 → ACK 4 → resend **5–7**. |
| **SACK** | Optional (chosen at connection setup). Receiver reports **non-contiguous received blocks** after a loss. Needs **8 bytes** of optional header. |
| **Congestion control** ★ | Flow control = **receiver's** capacity; congestion control = **network's** capacity. Most common loss cause: **overwhelmed routers**. |
| **Rate adjustment** | Increases **by two every RTT** up to a **threshold**; then **+1 per RTT**; on ACK timeout or triple ACK, **threshold is cut in half**. |

### In-class question ★
**TCP sequence numbers…** (A. always start at 0, B. always start at 1, C. start with a random number, D. TCP has no sequence numbers)
→ **C — start with a random number.**

---

## 2. The socket interface ★

- A **socket** is a **software endpoint** enabling **two-way communication** between a server and a client over a network.
- Sockets can use **TCP or UDP**.
- Fundamental for **web servers, chat systems, multiplayer games**.
- A socket is a **low-level interface** to the network. It's the **interface to the Transport layer**.
- Available in **Python, Java, C, C++**, and more.
- Higher-level network software (HTTP libraries, etc.) generally has **sockets deep inside**.

### Socket types
| Language | TCP client | TCP server | UDP |
|---|---|---|---|
| **Java** | `java.net.Socket` | `java.net.ServerSocket` | `java.net.DatagramSocket` (sends `DatagramPacket`) |
| **Python** | `socket.socket(AF_INET, SOCK_STREAM)` | same, with `bind`/`listen`/`accept` | `socket.socket(AF_INET, SOCK_DGRAM)` |

- Python: `import socket`. **`SOCK_STREAM` = TCP**, **`SOCK_DGRAM` = UDP**.

### Socket methods ★ (same names in Java, Python, C++)
`socket()` · `bind()` · `listen()` · `accept()` · `connect()` · `send()` · `recv()` · `close()`

### Socket program flow ★ (slide diagram)
```
   CLIENT SIDE                 SERVER SIDE
   socket                      socket
     ↓                           ↓
   connect                     bind
     ↓                           ↓
   send  ←──┐                  listen
     ↓      │                    ↓
   recv ────┘                  accept  ←────────┐
     ↓                           ↓              │
   close                       recv  ←──┐       │
                                 ↓      │       │
                               send ────┘       │
                                 ↓              │
                               close ───────────┘
```
- **Server:** `socket → bind → listen → accept → (recv ⇄ send loop) → close → back to accept` for the next client.
- **Client:** `socket → connect → (send ⇄ recv loop) → close`.
- **Reading/writing to a socket is identical to reading/writing a file** (Java `Socket` creates streams used exactly like file streams).

---

## 3. Example: TCP client and server

**Scenario:** the server listens on a port and waits. The **client** sends a message, receives a reply, displays it. The **server** receives the message, sends a reply, displays the message.

### Python server ★
```python
import socket
HOST = "127.0.0.1"   # this host's IP address, localhost
PORT = 47600         # port where the program will listen

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.bind((HOST, PORT))
    sock.listen()
    conn, addr = sock.accept()
    with conn:
        print(f"Connected by {addr}")
        data = conn.recv(1024)
        print(f"Received {data!r}")
        conn.sendall(data)
```

| Line | What it does |
|---|---|
| `socket.socket(AF_INET, SOCK_STREAM)` | Creates the server socket. **`AF_INET`** = Internet (IPv4) addresses; **`SOCK_STREAM`** = TCP byte stream. |
| `sock.bind((HOST, PORT))` | Tells the system this program uses that **port on that host**. The host must be **this** host. |
| `sock.listen()` | Tells the system it will **wait for a client**. |
| `conn, addr = sock.accept()` | **Blocks** until a client connects. Returns a **new socket object** (`conn`) connected to the client, plus the client's address. |
| `conn.recv(1024)` | Reads **up to 1024 bytes**. |
| `conn.sendall(data)` | Sends **all** bytes, even if they can't go in one `send`. |

### Python client ★
```python
import socket
HOST = "whatever.ncat.edu"   # server's hostname or IP address
PORT = 47600                 # port used by the server

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.connect((HOST, PORT))
    sock.sendall(b"Hello, world\n")
    data = sock.recv(1024)

print(f"Received {data!r}")
```
- Socket is created the **same way** as on the server.
- **`connect((HOST, PORT))`** names the server host and port.
- Data is **bytes** (`b"..."`), not `str`. Use `.encode()` / `.decode()` to convert.

### Java server ★
```java
public class Tserver {
  final static int serverPort = 47600;
  public static void main(String[] args) {
    java.net.ServerSocket sock = null;
    java.net.Socket       clientSocket = null;
    java.io.PrintWriter   pw = null;
    java.io.BufferedReader br = null;
    try {
      sock = new java.net.ServerSocket(serverPort);
      clientSocket = sock.accept();
      pw = new java.io.PrintWriter(clientSocket.getOutputStream());
      br = new java.io.BufferedReader(new java.io.InputStreamReader(
                                       clientSocket.getInputStream()));
      String msg = br.readLine();
      System.out.println("Message from the client >" + msg);
      pw.println("Got it!");
      pw.flush(); pw.close(); br.close(); sock.close();
    } catch (Throwable e) {
      System.out.println("Error!!");
    }
  }
}
```
- `new ServerSocket(port)` creates the socket **bound to the port** and **starts listening**.
- `accept()` **waits** for a client, then returns a **new `Socket`** bound to that client.
- The program gets the socket's **input and output streams** and **wraps** them in ordinary I/O objects.
- Note `pw.flush()` — the server's `PrintWriter` has **no auto-flush**, so it must flush manually (the client uses `true` for auto-flush).

### Java client ★
```java
public class Tclient {
  final static String serverName = "whatever.ncat.edu";
  final static int serverPort = 47600;
  public static void main(String[] args) {
    java.net.Socket sock = null;
    java.io.PrintWriter pw = null;
    java.io.BufferedReader br = null;
    try {
      sock = new java.net.Socket(serverName, serverPort);
      pw = new java.io.PrintWriter(sock.getOutputStream(), true);
      br = new java.io.BufferedReader(new java.io.InputStreamReader(
                                       sock.getInputStream()));
      pw.println("Message from the client");
      String answer = br.readLine();
      System.out.println("Response from the server >" + answer);
      pw.close(); br.close(); sock.close();
    } catch (Exception e) {
      System.out.println("Error!!");
    }
  }
}
```
- `new Socket(name, port)` takes the server's **Internet name or address** plus its **port**.

### Java I/O classes (slide) ★
| Class | Role |
|---|---|
| `java.io.InputStreamReader` | Reads **one character at a time** |
| `java.io.BufferedReader` | **Buffers** characters from an input stream; lets you **read lines** (`readLine()`) |
| `java.io.OutputStream` | **Abstract** class that sends **bytes** |
| `java.io.PrintWriter` | Methods to **write various forms of data** (`println`, etc.) |

### Differences between Java and Python ★
| Python | Java |
|---|---|
| **Explicitly calls `connect`** | Connection is **created when you create the `Socket` object** |
| No required exception handling | **Must catch possible exceptions** |
| `bind()` then `listen()` as separate calls | Creating a **`ServerSocket` binds it to the port** on this host **and initiates listening** |

---

## 4. Higher-level network access
- Sockets can do **anything** the network can do; **all network software eventually uses sockets**.
- Higher-level constructs are often **easier** than raw sockets.

**Read a URL in Java** (`URI` → `URL` → `openStream()` → wrap in readers → read lines in a loop):
```java
java.net.URI uri = new java.net.URI("https://comp476.online/getMyScore.html");
java.net.URL webFile = uri.toURL();
java.io.InputStream inFile = webFile.openStream();
java.io.BufferedReader bufRead = new java.io.BufferedReader(
                                   new java.io.InputStreamReader(inFile));
String html = bufRead.readLine();
while (html != null) { System.out.println(html); html = bufRead.readLine(); }
bufRead.close();
```

**Read a URL in Python** (built-in library):
```python
import urllib.request
url = "https://comp476.online/getMyScore.html"
with urllib.request.urlopen(url) as response:
    html_content = response.read()
    print(html_content.decode('utf-8'))   # pages are often UTF-8
```

---

## 5. "Where am I?" — finding your own address
- A socket program needs the **IP name or address of the other computer**.
- You pass it to **`Socket(...)` in Java** or **`connect(...)` in Python**.
- A student **server** program may want to **display its own address** so the client knows where to connect.

```java
// Java
java.net.InetAddress localHost = java.net.InetAddress.getLocalHost();
String ipAddress = localHost.getHostAddress();
```
```python
# Python
import socket
hostname = socket.gethostname()
private_ip = socket.gethostbyname(hostname)
```
- Java also has **`Inet4Address`** and **`Inet6Address`** for a specific IP version.

---

## 6. Web servers and threads ★
- A Java web server does `sock = new ServerSocket(serverPort); clientSocket = sock.accept();`.
- **`accept` creates a new socket** to communicate with the client that just connected (slide: "the new socket has a new port number").
- The **original server socket keeps listening on port 80**.
- Typical design:
  1. Main thread **waits on `accept`**.
  2. Client connects → server **creates a new thread** for that client.
  3. Main thread **goes back to waiting** for the next client.
  4. The new thread **handles all requests from that client**, and **terminates when the client is done**.

> ⚠️ **Slide vs. textbook:** the slide says the accepted socket gets "a new port number." In real TCP, the accepted connection still uses the **server's same local port (80)**; connections are told apart by the **full 4-tuple (client IP, client port, server IP, server port)**. It's the **client** that has the temporary port. The point to remember either way: **one listening socket + one new socket per client**.

---

## 7. UDP sockets ★

**Java:** `java.net.DatagramSocket` is the UDP socket; it sends/receives **`java.net.DatagramPacket`** objects. Each packet holds the **bytes** plus the **destination address and port**.

**Java UDP server**
```java
byte[] buffer = new byte[BUFSIZE];
java.net.DatagramSocket udpSock = new java.net.DatagramSocket(PORT);
java.net.DatagramPacket pkt = new java.net.DatagramPacket(buffer, BUFSIZE);
System.out.println("waiting");
udpSock.receive(pkt);                       // blocks until a packet arrives
System.out.println("received packet");
String stuff = new String(pkt.getData());
System.out.println("msg>" + stuff);
udpSock.close();
```
- Server socket is created **with the port** so it receives there.

**Java UDP client**
```java
java.net.DatagramSocket udpSock = new java.net.DatagramSocket();   // no port needed
// ... read server IP and message from keyboard ...
buffer = msg.getBytes();
java.net.InetAddress hostAddr = java.net.InetAddress.getByName(server);
java.net.DatagramPacket pkt = new java.net.DatagramPacket(
        buffer, 0, buffer.length, hostAddr, PORT);
udpSock.send(pkt);
```

**Python UDP client** (slide title typo'd "UPD")
```python
import socket
UDP_IP = "127.0.0.1"
UDP_PORT = 47600
message = "Hello, UDP from Python!"
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)   # DGRAM = UDP
try:
    sock.sendto(message.encode('utf-8'), (UDP_IP, UDP_PORT))
finally:
    sock.close()          # ensure the socket is closed
```

### TCP vs. UDP sockets — what changes
| | TCP socket | UDP socket |
|---|---|---|
| Python type | `SOCK_STREAM` | `SOCK_DGRAM` |
| Setup | `connect` / `listen` + `accept` | **No connection**; just send |
| Send call | `send` / `sendall` | **`sendto(data, (ip, port))`** — address with every packet |
| Java classes | `Socket`, `ServerSocket` | `DatagramSocket`, `DatagramPacket` |
| Reliability | Reliable stream | Packets may be lost/reordered |

*(Not on slides: a Python UDP server is `bind((HOST, PORT))` then `data, addr = sock.recvfrom(1024)`.)*

---

## 8. Course logistics
- Slide reminder: **check voter registration at vote.gov**; register before **Nov 3**; **same-day registration is possible during early voting but not on election day**; early voting runs through **Sat Oct 31**.

---

## 9. Quick fact sheet

| Concept | Key fact |
|---|---|
| Socket | Software endpoint for two-way communication; **interface to the Transport layer** |
| Socket methods | `socket, bind, listen, accept, connect, send, recv, close` |
| TCP vs. UDP in Python | `SOCK_STREAM` vs. `SOCK_DGRAM` |
| `AF_INET` | Use Internet (IPv4) addresses |
| Server flow | socket → bind → listen → **accept** → recv/send → close → (accept again) |
| Client flow | socket → **connect** → send/recv → close |
| `accept()` | **Blocks**; returns a **new socket** + client address; the original keeps listening |
| `sendall` | Sends **all** bytes even if it takes multiple sends |
| `recv(n)` | Reads **up to n** bytes |
| Java TCP | `Socket` (client), `ServerSocket` (server; **binds + listens on creation**) |
| Java UDP | `DatagramSocket` + `DatagramPacket` (bytes + address + port) |
| Java vs. Python | Java: no explicit `connect`, **must catch exceptions** |
| Java I/O | `BufferedReader` → `readLine()`; `PrintWriter` → `println()`; **flush** if no auto-flush |
| Web server | One listening socket; **new thread per client** |
| Own address | Java `InetAddress.getLocalHost().getHostAddress()`; Python `socket.gethostbyname(socket.gethostname())` |
| Sequence number start | **Random** |
