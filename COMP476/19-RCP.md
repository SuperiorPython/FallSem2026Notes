# COMP476 — Networked Computer Systems
## Lecture 19 — Remote Procedure Calls (slides: "Remote Procedure Calls," Mon Oct 26, 2026)

**Relationship to Lecture 18:** this deck re-covers the **HTTP → RPC → Web API → Base64** sequence from Lecture 18 in a tighter order, then adds the **Online API Assignment**. Items marked 🆕 are new or reframed vs. L18; everything else is review (see Lecture-18-HTTP-RPC-Notes.md for the long-form HTTP detail).

---

### 1. HTTP recap (review)
- **HTTP** = application layer protocol, generally on **TCP and IP**. **HTTPS** = HTTP **+ TLS** (security covered in a couple of weeks).
- Main **request–response (client–server)** protocol for web documents. Other high-level web protocols: **FTP**, **Telnet**.
- Requests/responses are **ASCII**, human readable. Request ends with a **blank line**.
```
Method  filename  HTTP/1.1 <CR><LF>
options           <CR><LF>
<CR><LF>
```
```
GET /mypage.html HTTP/1.1
HOST: comp476.online
```
- **`HOST` is required** in 1.1.

| Method | Purpose |
|---|---|
| **GET** | Get a file |
| **HEAD** | Get **info about** a file |
| **POST** | **Send** info to server |
| **PUT** | Send a file to be **stored** |
| **DELETE** | Delete a file |
| **OPTIONS** | Available server options |
| **TRACE** | Loop-back of the request |

**Status codes:** **1xx** Informational · **2xx** Success (**200 OK**) · **3xx** Redirection (**301 Moved Permanently**, **304 Not Modified**).

**Response:** `HTTP/1.1 statuscode reason` → response options → blank line → file contents. Real example headers: `Server`, `Date`, `Content-Type`, `Vary`, `Last-Modified`, **`Cache-Control: max-age=500, public, proxy-revalidate`**, `Expires`.

**Multiple files per page:** page = HTML + embedded files; HTML fetched first, **each file fetched separately**. **HTTP/1.1 keep-alive** keeps the connection open to transfer multiple files (client: socket → connect port **80** → GET → read header + contents; then GET each graphic on the **existing connection**).

---

## RPC

### 2. Why RPC? (Motivation) ★
- **Sockets** provide **end-to-end communication between processes**, but:
  - Require **manual message packing, sending, parsing**
  - **Limited by complexity** for distributed systems
- Sockets are powerful but **low-level**; developers **repeat communication patterns** and are **prone to implementation mistakes** → need a **higher-level abstraction**.

### 3. Function execution → RPC ★
- Normal `x = func(z)`: `func` runs on the **same computer**, in a different part of memory, using the **main program's thread**.
- **Middleware** can execute the procedure on a **remote computer**.
- **RPC** = a function call **executed on a remote machine that appears local**; a **paradigm for writing distributed programs**; more organized and high-level than socket send/receive.

**Advantages**
- Looks like a **local function call**; similar to conventional programming.
- Procedure calling has **well-understood semantics**.
- **Simplifies access to remote systems**; handles communication **under the hood**.

**Reasons for remote procedures**
- Data **centrally located** (databases used by many programs)
- **Resource sharing** of peripherals
- Function needs **special hardware** (e.g., vector processor)
- Needs a **more powerful processor**

### 4. RPC operation ★ (step order is testable)
1. **Main program calls the client stub.**
2. Client stub **marshals** (copies) the parameters into a **communication packet**.
3. Client stub **sends an RPC request** to the server.
4. **Server** calls the desired function, passing the parameters from the packet.
5. When the function returns, the server **sends the results** to the client stub.
6. Client stub **returns the results to the caller**.

*(Slide figure: **Logical** flow looks like a direct call; **Actual** execution flow goes through both stubs and the network.)*

**Construction:** main calls `int myfunc(float x);`
- **Client stub** named `myfunc` — does **no computation**, sends the parameter to a remote system.
- **Server stub** receives data, calls the **actual `myfunc`**, sends back results.

### 5. Parameters ★
**Input (caller → function)**
| Type | Handling |
|---|---|
| Basic datatype | **Pass by value**, copy to server |
| Arrays / structures | **Copy** to server (may be a lot of data) |
| Pointer to single datatype | **Copy the data item** |
| Pointers in general | **Not allowed** |
- **Java:** data must be **serializable**.

**Output (function → caller)**
- C uses pointers for outputs: `x = func(&z);`
- **Data, not addresses, is copied** back; **return value** copied too.

### 6. 🆕 Simple RPC in Python (slide code)
**Server**
```python
import socket, json

def add(x, y): return x + y

s = socket.socket()
s.bind(('localhost', 5000))
s.listen(1)
while True:
  conn, _ = s.accept()
  data = json.loads(conn.recv(1024).decode())
  result = add(*data["params"])
  conn.send(json.dumps({"result": result}).encode())
```
**Client**
```python
import socket, json

def add(x, y):
     req = {"method": "add", "params": [x, y]}
     s = socket.socket()
     s.connect(('localhost', 5000))
     s.send(json.dumps(req).encode())
     response = json.loads(s.recv(1024).decode())
     print(response["result"])
```
- The client's `add` is the **client stub**: it **marshals** the call as **JSON** (`method` + `params`), sends it over a socket, and unmarshals the `result`.
- The server loop is the **server stub**: **accept → decode JSON → call `add(*params)` → send JSON result**.
- `*data["params"]` unpacks the list into arguments.
- **Port 5000**, `localhost`; `recv(1024)` caps a message at 1024 bytes (fine for a toy example).

### 7. Execution environment
- **No global variables**
- **No environment variables**
- **No access to files on the client computer**

### 8. Exception handling — delivery semantics ★
| Semantics | Behavior | Notes |
|---|---|---|
| **No retry** | Never resend | RPC may not work |
| **At-least-once** | **Keep sending until it gets there** | Needs **idempotent** functions |
| **At-most-once** | Retry, but **server must filter repeats** | |

### 9. Synchronization ★
- **Normal RPC = synchronous:** caller **waits** for completion; **no parallelism** (logical thread of execution moves to another computer and back).
- **Asynchronous RPC:** called function runs **in parallel** with the main program.
  - **Call without reply:** stub **returns after sending**; good for **void** functions (e.g., **print**).
  - **Callbacks:** RPC returns **before** the result is complete; server **calls a completion function in the client** when done. Supports **event-driven programming**; generally **multiple threads** in the caller.

### 10. Automatic stub generation & IDL ★
- Systems **auto-generate client and server stubs** (e.g., **EasyRPC**).
- **IDL (Interface Definition Language)** defines the interface like a **function prototype**:
```
int funcabc(
       [in]         float   x,
       [out]        long   *y,
       [in out]     double *z
       );
```
- **DCE flow:** **IDL source → RPC generator → client stub + server stub**; stubs + **RPC gen library** + **compiler** combine with the **main program** (→ client program) and the **function** (→ server program).
- **Common RPC frameworks:** **gRPC** (Google), **XML-RPC**, **JSON-RPC**, **Thrift** (Apache), **RMI** (Java).

**Pros and cons ★**
| Advantages | Limitations |
|---|---|
| Simplifies distributed programming | **Hidden network failures** |
| Abstracts networking details | **Latency not obvious** |
| Supports multiple languages | **Harder debugging** |
| | **Version compatibility** issues |

### 11. TCP or UDP for RPC?
| UDP | TCP |
|---|---|
| **Simple idempotent** functions | When **reliable transfer** is required |
| **Simpler, more efficient** | **Network problems resolved by TCP** |
| **You must check for transmission errors** | Probably best if UDP would need **multiple transmissions** |

---

## Web APIs

### 12. API basics ★
- **API** = software interface **offering a service to other software**; generally executes on a **remote computer**.
- **API specification** = document/standard describing how to build the connection/interface.
- **Web-based APIs** generally use **HTTP**; responses usually **JSON** or **XML**.
- **About half of Internet traffic is not intended to be read directly by humans.**
- **Marshalling for a web API:** the stub creates a **URI with the specified parameters**; **encoding of data** may be necessary.
- **Human viewable:** build the URL, paste it in a browser, read the JSON/XML.

### 13. Web API call examples
**Python** — `urllib.request.urlopen(url + parm)` → `response.read()` → `.decode('utf-8')` → use the result.
```python
import urllib.request
myParm = "inputParm"
url = "https://comp476.online/getMyScore.html?" + myParm
with urllib.request.urlopen(url) as response:
  html_content = response.read()
  decoded_content = html_content.decode('utf-8')
```
**Java currency converter** — `main` reads two currency codes, calls `getConversion(mon1, mon2)`. `getConversion` builds a **URI/URL** to an exchange-rate API (`.../latest/<cur1>`), opens a stream, reads the JSON into a String, uses **`indexOf(cur2)`** and **`substring`** up to the next comma to pull out the rate. (Crude string parsing; a JSON library is the proper way.)

### 14. Security & business use
- Many web APIs require a **code (API) key**; parameters may be **encrypted** in addition to **HTTPS**.
- Big companies expose APIs to customers' systems: **product & service catalog, product ordering, payment systems**.

---

### 15. Base64 encoding ★
- **Problem:** binary data sent over networks expecting **regular characters**; some byte values are punctuation/control characters (**return, comma, ETX, NAK, null**) that **confuse systems**. *(256 byte values vs. 92 keyboard characters.)*
- **Base64** converts **any data** into a string of **alphanumeric characters**.
- ⚠️ **Encoding ≠ encryption** — looks random but **there is no key**.

**Binary → characters**
- **3 bytes (24 bits)** → **four 6-bit values**; 2⁶ = **64** values: **26 upper + 26 lower + 10 digits + `+` and `/`**.
- Slide example: `X`, `CR`, `LF` → `010110 000000 110100 001010` → **`W A 0 K`**.

**Padding with `=`**
| Input | Len | Output | Len | Padding |
|---|---|---|---|---|
| `light work.` | 11 | `bGlnaHQgd29yay4=` | 16 | **1** |
| `light work` | 10 | `bGlnaHQgd29yaw==` | 16 | **2** |
| `light wor` | 9 | `bGlnaHQgd29y` | 12 | **0** |
| `light wo` | 8 | `bGlnaHQgd28=` | 12 | **1** |
| `light w` | 7 | `bGlnaHQgdw==` | 12 | **2** |
- Length not a multiple of 3 → `=` pads. **1 leftover byte → `==`**, **2 leftover bytes → `=`**, none → no padding.
- Example: `COMP476 Networked Computer Systems` → `Q09NUDQ3NiBOZXR3b3JrZWQgQ29tcHV0ZXIgU3lzdGVtcw==`
- Rule of thumb: a random-looking string of letters/numbers is **probably Base64**.

---

### 16. 🆕 Online API Assignment (due soon)
- Posted on **Canvas**: access **three online APIs**.
- Get your assignment details from **`comp476.online/APIasg/XXXX.pdf`**, where **XXXX = your Base64-encoded magic number** (use Base64 from §15 to build the URL).
- **Due by noon, Wednesday, November 4, 2026.**
- **Upload your solution's source code to Canvas.**

*(Note: that date has passed as of today, Nov 18, 2026 — kept here for the record of what the lecture said.)*

---

### 17. Quick fact sheet
| Concept | Key fact |
|---|---|
| HTTP layer | **Application**, over **TCP/IP**; HTTPS = HTTP **+ TLS** |
| Required HTTP/1.1 header | **Host** |
| Request terminator | **Blank line** |
| Keep-alive | Reuse the **same connection** for multiple files |
| Why RPC | Sockets are **low-level**: manual pack/send/parse |
| RPC steps | main → client stub → **marshal** → server → function → result → stub → caller |
| Pointers in RPC | **Not allowed** — copy data, not addresses |
| Execution environment | **No globals, no env vars, no client files** |
| Retry semantics | No retry / **at-least-once** (idempotent) / **at-most-once** (filter repeats) |
| Sync vs async | Sync **waits**; async = **call w/o reply** or **callback** |
| IDL tags | `[in]`, `[out]`, `[in out]` |
| RPC frameworks | **gRPC, XML-RPC, JSON-RPC, Thrift, RMI** |
| RPC limits | **Hidden failures, latency, debugging, versioning** |
| UDP vs TCP | UDP = simple **idempotent**; TCP = **reliable** |
| Web API format | **JSON / XML** over **HTTP**; marshalling = **URI with params** |
| Base64 | **3 bytes → 4 chars**, 64-symbol alphabet, `=` padding, **not encryption** |
