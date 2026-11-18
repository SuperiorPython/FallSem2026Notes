# COMP476 — Networked Computer Systems
## Lecture 18 — HTTP and RPC (slides: "HTTP and RPC," Wed Oct 21, 2026; reading 6.5 — Online APIs)

**What's new vs. Lecture 16:** we're back at the top of the stack. This lecture covers **HTTP** (history, URLs, HTML, static/dynamic/active documents, request/response format, status codes, caching, keep-alive), then **RPC** (stubs, marshalling, failure semantics, IDL), then **web APIs** and **Base64**.

---

### 1. Where HTTP sits
```
Application    ← you are here (HTTP)
Transport      (TCP)
Internet       (IP)
Logical Data Link
Media Access Control
Physical
```
- **HTTP** = HyperText **Transport/Transfer** Protocol — an **application layer** protocol, generally on top of **TCP and IP**.
- **HTTPS** = HTTP **plus TLS** (security covered in a couple of weeks).

### 2. History & standards
- HTTP and HTML created in **1989 at CERN** (European Center for Nuclear Research) by **Tim Berners-Lee** (also created the **URL system**). Knighted by Queen Elizabeth II in **2004**. Both his parents were early computer scientists on the **Mark 1**.
- **HTTP/1.0** finalized and documented in **1996**.
- Defined by **RFC 2616** and others from the **IETF**.
- Originally for **hypertext**; now a **general application layer protocol** for all kinds of data.
- **Firewalls** usually allow **port 80 (HTTP) and 443 (HTTPS)** → apps that want to dodge firewall problems tunnel over HTTP.

### 3. Client–server paradigm
- **Server** waits at a **known address**; **client** contacts it.
- The **World Wide Web** is a client–server system over TCP/IP.

---

### 4. URI vs. URL ★
- **URI** identifies an abstract or physical resource.
- **URL** = a URI that also gives the **location** and a **mechanism for retrieving** it.
- **Every URL is a URI** (URLs ⊂ URIs).

**URL anatomy ★**
```
http://john.doe:password@www.example.com:123/forum/questions/?tag=networking&order=newest#top
└scheme┘ └────── authority ──────────────┘└── path ──────┘└────── query ──────────────┘└fragment┘
         userinfo        host        port
```
| Part | Example |
|---|---|
| **scheme** | `http` |
| **authority** = userinfo + host + port | `john.doe:password@www.example.com:123` |
| **path** | `/forum/questions/` |
| **query** | `?tag=networking&order=newest` |
| **fragment** | `#top` |

**Common schemes**
| Scheme | Used for |
|---|---|
| `http` | Hypertext (HTML) |
| `https` | Hypertext with security |
| `ftp` | FTP |
| `file` | Local file |
| `mailto` | Sending email |
| `rtsp` | Streaming media |

### 5. Hypertext, hypermedia, HTML
- The Web is a **distributed hypermedia system** with interactive access.
- **Hypermedia** extends hypertext with elements **other than text**.
- **Distributed** = spread over many independent systems → **links can break**. (A non-distributed/closed system, e.g. one computer, can verify every link.)
- **HTML** gives **general guidelines** for display; the **browser picks the details** → two browsers may render the same page differently. Current version on slides: **HTML 5.2**.
- **Tags:** `<TAGNAME>` to start, `</TAGNAME>` to end.
- **Link:** `<A HREF="linkname.html"> Display text </A>`
- **Image:** `<IMG SRC="imagename.png">`

---

### 6. Types of web documents ★
| Type | What happens | Example |
|---|---|---|
| **Static** | Same **file** sent on every request; content fixed when author wrote it | Static web pages, FTP files |
| **Dynamic** | **Program at the server** builds the document per request; can differ each time | CGI, PHP, Java, JavaScript, C++ server programs |
| **Active** | **Program sent from server, runs on the client** | JavaScript in HTML |

**Dynamic documents / CGI**
- Doesn't exist in predefined form; the server **runs a program** on each request and returns its **output**.
- **CGI (Common Gateway Interface)** = standard for how a program interacts with the server. **Any language, any output type.**
- CGI header tells the type, e.g. `Content-type: text/html` or `Location: /new/file.txt`.
- **Parameters passed in environment variables.**
- **Forms** send field values as `fieldname=value,field2=value2`.

**Active documents**
- Scripts embedded in HTML; can calculate, access the server, etc.
- **JavaScript** is the most popular HTML scripting language.
- **JavaScript ≠ Java.** Downloaded **Java bytecode** was supposed to run in a **sandbox**, but users worried about running unknown programs → fell out of favor.

### 7. Browser architecture & file base
- Browsers are **more complex** than servers (servers do simple, repetitive work).
- Browser = **controller + set of clients + set of interpreters**.
- **Web base:** one server directory is the root for web pages; requests are rooted there.
  - No file specified → **`index.html`** returned.
  - **Executable** requested → it's **run** and its **standard output** goes to the browser.

---

### 8. HTTP request format ★
- Request/response are **ASCII text**, human readable.
- Request is **terminated by a blank line** (two end-of-line characters).
```
Method  filename  HTTP/1.1 <CR><LF>
options           <CR><LF>
<CR><LF>
```
**Example**
```
GET /mypage.html HTTP/1.1
HOST: comp476.online
```
- Asks the server to send `mypage.html`; browser says it uses **HTTP 1.1**.
- **`HOST` is required** in 1.1.

**HTTP methods ★**
| Method | Purpose |
|---|---|
| **GET** | Get a file from the server |
| **HEAD** | Get **info about** a file (no body) |
| **POST** | **Send** information to the server |
| **PUT** | Send a file to be **stored** on the server |
| **DELETE** | Delete a file on the server |
| **OPTIONS** | Ask for available server options |
| **TRACE** | Loop-back of the request message |

### 9. Server response & status codes ★
```
HTTP/1.1 statuscode reason
response options
<blank line>
file contents
```
| Range | Meaning | Examples |
|---|---|---|
| **1xx** | Informational | |
| **2xx** | Success | **200 OK** |
| **3xx** | Redirection | **301 Moved Permanently**, **304 Not Modified** |
| **4xx** | **Client** error | **400** Bad Request, **401** Unauthorized, **402** Payment Required, **403** Forbidden, **404** Not Found, **410** Gone |
| **5xx** | **Server** error | **500** Internal Server Error |

### 10. Reading a real request (Chrome → comp476.online)
| Header | Meaning |
|---|---|
| `Host` | Server being contacted (required) |
| `Sec-Ch-Ua-Mobile: ?0` | **Desktop** (`?1` = mobile) |
| `Sec-Ch-Ua-Platform: "Windows"` | User's **OS** |
| `Accept-Language: en-US,en;q=0.9` | Preferred languages |
| `User-Agent` | Browser/OS identification string |
| `Accept: text/html,…` | **File formats** the device can display |
| `Accept-Encoding: gzip, deflate, br` | **Compression** methods supported |
| `Connection: keep-alive` | Keep the TCP connection open |

**Quality values (`q`) ★**
- Range **0 to 1**; **1 = highest preference**; **0 = undesirable**.
- `en-US;q=0.9, en;q=0.8` → U.S. English preferred, general English acceptable.

### 11. Response headers & caching ★
```
HTTP/2 200 OK
Server: openresty
Date: Tue, 21 Oct 2025 19:28:26 GMT
Content-Type: text/html
Vary: Accept-Encoding
Last-Modified: Sat, 18 Oct 2025 21:37:33 GMT
Cache-Control: max-age=500, public, proxy-revalidate
Expires: Tue, 21 Oct 2025 19:36:46 GMT
```
- Browsers **cache** received files on local disk.
- If a cached copy exists, the browser sends a GET with **`If-Modified-Since: <date>`**.
  - Not modified → server replies **304 Not Modified** with an **empty body**.
- **`Cache-Control: max-age=500`** → file is **outdated after 500 seconds**.
- **`Cache-Control` overrides `Expires`.**

### 12. Multiple files per page, keep-alive ★
- A page = the **HTML file + embedded files**; browser gets the HTML first, which names the rest; **each file fetched separately** via HTTP (often from **different servers**).
- **`Connection: keep-alive`** → server **keeps the TCP connection open**; further requests **reuse** it.

**HTTP/1.1 operations**
| Client | Server |
|---|---|
| Create a socket | |
| Connect to server at **port 80** | |
| Send **GET** for the file | Send a **header** describing the file, then the **contents** |
| (keep-alive) send **GET** on the **existing connection** for each graphic | Send header + contents of the graphic over the existing connection |

- **`favicon.ico`** is often requested even when the HTML never mentions it: tiny image shown on the **browser tab** next to the title.

---

## RPC

### 13. Why RPC? ★
- **Sockets:** end-to-end process communication, but **low-level** — manual message packing, sending, parsing; complex for distributed systems.
- Developers **repeat communication patterns** and make mistakes → need a **higher-level abstraction**.

### 14. What an RPC is ★
- Normal call `x = func(z)`: `func` runs on the **same computer** using the main program's thread.
- **RPC** = a function call **executed on a remote machine that looks local**. **Middleware** runs the procedure remotely.
- A paradigm for **distributed programs**; higher-level than socket send/receive.

**Advantages**
- Looks like a **local function call**; similar to conventional programming; **well-understood call semantics**.
- **Simplifies access to remote systems**; handles communication **under the hood**.

**Why run remotely?**
- Data **centrally located** (shared databases).
- **Resource sharing** of peripherals.
- Needs **special hardware** (e.g., vector processor).
- Needs a **more powerful processor**.

### 15. RPC operation ★ (step order is testable)
1. **Main program calls the client stub.**
2. Client stub **marshals** (copies) the parameters into a **communication packet**.
3. Client stub **sends an RPC request** to the server.
4. **Server stub** calls the desired function with the parameters from the packet.
5. When the function returns, the server **sends the results** to the client stub.
6. Client stub **returns the results to the caller**.

**Construction example** — main calls `int myfunc(float x);`
- **Client stub** named `myfunc`: does **no computation**, just sends the parameter to the remote system.
- **Server stub** receives the data, calls the **real `myfunc`**, sends back the result.

### 16. Parameters ★
**Input (caller → function)**
| Type | Handling |
|---|---|
| Basic datatype | **Pass by value**, copy to server |
| Arrays / structures | **Copy** to server (may be a lot of data) |
| Pointer to a single datatype | **Copy the data item** |
| Pointers in general | **Not allowed** |
- In **Java**, data must be **serializable**.

**Output (function → caller)**
- C uses pointers for outputs: `x = func(&z);`
- **Data, not addresses, is copied** back from the server; **return value copied** too.

### 17. RPC execution environment
- **No global variables**, **no environment variables**, **no access to files on the client computer** (the function runs on the server's machine).

### 18. Exception handling — delivery semantics ★
If the stub's first attempt to send fails:
| Semantics | Behavior | Notes |
|---|---|---|
| **No retry** | Never resend | RPC may not work |
| **At-least-once** | **Keep sending until it gets there** | Needs **idempotent** functions (safe to run twice) |
| **At-most-once** | Retry, but **server filters repeats** | |

### 19. Synchronization
- **Normal RPC = synchronous:** caller **waits** until the function completes; **no parallelism** (the logical thread of execution moves to the other computer and back).
- **Asynchronous RPC:** called function runs **in parallel** with the main program.
  - **Call without reply:** stub **returns right after sending**. Good for **void** functions (e.g., a **print** function).
  - **Callbacks:** RPC returns **before** the result is ready; server **calls a completion function in the client** when done. Supports **event-driven programming**; generally needs **multiple threads** in the caller.

### 20. Automatic stub generation & IDL ★
- Systems can **auto-generate client and server stubs** (e.g., **EasyRPC**).
- **IDL (Interface Definition Language)** defines the function interface, like a **function prototype**:
```
int funcabc(
    [in]      float   x,
    [out]     long   *y,
    [in out]  double *z
);
```
- **Flow (Distributed Computing Environment):** IDL source → **RPC generator** → **client stub** + **server stub**. Each is compiled with the **RPC gen library** along with the **main program** (→ client program) and the **function** (→ server program).

**Common RPC frameworks:** **gRPC** (Google), **XML-RPC**, **JSON-RPC**, **Thrift** (Apache), **RMI** (Java).

**Pros and cons ★**
| Advantages | Limitations |
|---|---|
| Simplifies distributed programming | **Hidden network failures** |
| Abstracts networking details | **Latency not obvious** |
| Supports multiple languages | **Harder debugging** |
| | **Version compatibility** issues |

### 21. TCP or UDP for RPC?
| UDP | TCP |
|---|---|
| Fine for **simple idempotent** functions | Use when **reliable transfer** is required |
| **Simpler, more efficient** | **TCP resolves network problems** |
| **You must check for transmission errors** | Probably best if UDP would need **multiple transmissions** |

### 22. Simple RPC in Python (slide code, summarized)
- **Server:** create a socket → `bind(('localhost', 5000))` → `listen(1)` → loop: `accept()`, `json.loads(conn.recv(1024).decode())`, call `add(*data["params"])`, send back `json.dumps({"result": result})`.
- **Client:** build `req = {"method": "add", "params": [x, y]}` → connect to port **5000** → `send(json.dumps(req))` → `json.loads(s.recv(1024))` → print `response["result"]`.
- Here **JSON is the marshalling format**; the `add` function on the client is the **stub**.

---

## Web APIs

### 23. API basics ★
- **API** = a software interface offering a service to **other software**; usually executes on a **remote computer**.
- **API specification** = document/standard describing how to build the connection/interface.
- **Web-based APIs** use **HTTP**; responses usually **JSON** (JavaScript Object Notation) or **XML** (Extensible Markup Language).
- **About half of Internet traffic is not intended to be read directly by humans.**
- **Marshalling for a web API:** the stub builds a **URI with the parameters**; **data encoding** may be needed.
- **Human-viewable:** you can build the URL and paste it in a browser to inspect the JSON/XML.
- **Security:** many require an **API key**; parameters may be **encrypted** in addition to HTTPS.
- **Business use:** companies expose APIs for customers' systems — **product/service catalog, ordering, payments**.

**Python web API call (pattern):** `urllib.request.urlopen(url + parm)` → `response.read()` → `.decode('utf-8')` → process.

**Java currency example (pattern):** `getConversion(cur1, cur2)` builds a URL to an exchange-rate API (`.../latest/<cur1>`), reads the JSON as a string, **finds `cur2` with `indexOf`**, and **substrings** to the next comma to get the rate. (Crude parsing — a JSON library would be better.)

### 24. Base64 encoding ★
- **Problem:** binary data sent over networks expecting **regular characters** — some byte values look like **control/punctuation characters** (return, comma, ETX, NAK, null) and confuse systems.
- **Base64** converts **any data** into **alphanumeric text**.
- ⚠️ **Encoding ≠ encryption** — it looks random but there is **no key**.

**How it works**
- **3 bytes (24 bits)** → split into **four 6-bit values**.
- 2⁶ = **64** possible values: **26 uppercase + 26 lowercase + 10 digits + "+" and "/"**.
- Slide example: `X`, `CR`, `LF` (3 bytes) → 4 characters **`WA0K`**.
- Output is **4/3 as long** as the input.

**Padding with `=` ★**
| Input | Len | Output | Len | Padding |
|---|---|---|---|---|
| `light work.` | 11 | `bGlnaHQgd29yay4=` | 16 | **1** |
| `light work` | 10 | `bGlnaHQgd29yaw==` | 16 | **2** |
| `light wor` | 9 | `bGlnaHQgd29y` | 12 | **0** |
| `light wo` | 8 | `bGlnaHQgd28=` | 12 | **1** |
| `light w` | 7 | `bGlnaHQgdw==` | 12 | **2** |
- Input length not a multiple of 3 → `=` pads the end. (**1 leftover byte → `==`**, **2 leftover bytes → `=`**, 0 leftover → none.)
- Slide example: `COMP476 Networked Computer Systems` → `Q09NUDQ3NiBOZXR3b3JrZWQgQ29tcHV0ZXIgU3lzdGVtcw==`
- Rule of thumb: a string of letters/numbers that looks random is **probably Base64**.

---

### 25. Course logistics
- **Class Interest Survey** on **Canvas**, due before **7:00 pm Sunday, October 25, 2026**. Anonymous; answer honestly.

### 26. Quick fact sheet
| Concept | Key fact |
|---|---|
| HTTP layer / transport | **Application layer**, over **TCP/IP** |
| HTTP vs HTTPS | HTTPS = HTTP **+ TLS** |
| Firewall-friendly ports | **80** (HTTP), **443** (HTTPS) |
| Creator / year | **Tim Berners-Lee, 1989, CERN**; HTTP/1.0 in **1996** |
| URL vs URI | Every **URL is a URI**; URL adds location + retrieval method |
| URL parts | scheme, authority (userinfo, host, port), path, query, fragment |
| Document types | **Static** (same file), **Dynamic** (server program), **Active** (client program) |
| CGI | Standard for server programs; params via **environment variables** |
| Required HTTP/1.1 header | **Host** |
| Request terminator | **Blank line** |
| Cache | `If-Modified-Since` → **304**; `Cache-Control` **overrides** `Expires` |
| Keep-alive | Reuse the **same TCP connection** for multiple files |
| RPC steps | main → client stub → **marshal** → server stub → function → result back |
| Pointers in RPC | **Not allowed** (copy the data, not the address) |
| Retry semantics | No retry / **at-least-once** (idempotent) / **at-most-once** (filter repeats) |
| IDL param tags | `[in]`, `[out]`, `[in out]` |
| Web API formats | **JSON** or **XML** over **HTTP** |
| Base64 | **3 bytes → 4 chars**, 64-symbol alphabet, `=` padding, **not encryption** |
