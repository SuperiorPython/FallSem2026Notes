# COMP476 — Networked Computer Systems
## Lecture 20 — Email, FTP, and Text Messaging (slides: "Email," Wed Oct 28, 2026)

**Relationship to earlier lectures:** this deck is the **application layer** applied to messaging. It reuses **Base64** from Lecture 19 (review, see Lecture-19-RPC-Notes.md §15) and **HTTP** as a comparison point. Items marked 🆕 are new.

---

## Part 1 — Email

### 1. Overview ★
- **Email = a file with a specific header format**, transferred to the **mailbox** of the recipient.
- Two major parts: **sending** the email and **viewing/editing** the email.
- **Flow:** user interface (sender's computer) → **mail transfer program** (sender's computer) → **Internet** → **server** (recipient's computer) → **recipient's mailbox**.

### 2. Application-layer standards ★
| Standard | Defines |
|---|---|
| **X.400** | The **email file format** |
| **SMTP** (Simple Mail Transfer Protocol) | How mail is **sent between mail servers** |
| **IETF RFC 2821** | The SMTP standard |
| **POP** and **IMAP** | Protocols for **retrieving** mail from your mailbox |
| **RFC 822** | Standard for the **text message format** (header + body; see §6) |

### 3. Viewing email
- Programs that edit and view email: **Microsoft Outlook**, **Gmail** (web or app), **webmail**.
- These are **started by the user** and handle the **user interface**.
- Many email systems have a **web interface**.

### 4. Sending email
- A **background email daemon** sends and receives email files.
- Files are stored in the **system spool directory**, in a **file named for the user**.
- Email is a **store-and-forward** transmission system ★

### 5. Email file format ★
- Email is an **ASCII file**.
- The beginning holds **parameters** about the email, in the format:
```
parm-name:value<newline>
```
- The **header is separated from the message text by a blank line**.

### 6. Header fields ★
| Keyword | Meaning |
|---|---|
| **From** | Sender's address |
| **To** | Recipients' addresses |
| **Cc** | Addresses for **carbon copies** |
| **Date** | Date the message was sent |
| **Subject** | Topic of the message |
| **Reply-To** | Address to which the reply should go |
| **X-Charset** | Character set used (usually ASCII) |
| **X-Mailer** | Mail software used to send the message |
| **X-Sender** | Duplicate of sender's address |
| **X-Face** | **Encoded image** of the sender's face |

**What is carbon paper? (why "Cc")** — *carbon copy*:
- Typewriters: a metal letter hit an **ink ribbon** onto paper.
- **Carbon paper** (one side coated in **black graphite**) sat **between two pages**; the strike imprinted the letter on the **second page** → a **duplicate copy**.
- Side fact from the slide: the **QWERTY** layout was designed to be **inefficient** to avoid two keys hitting each other.

### 7. SMTP mail message format (RFC 822) ★
- **Header lines**, e.g., `To:`, `From:`, `Subject:` — **different from SMTP commands!**
- **Blank line**
- **Body** = the "message", **ASCII characters only**.
- *(Slide credit: Dr. Jinsheng Xu.)*

### 8. Email addresses ★
Two basic formats:
| Format | Example |
|---|---|
| `userid@computer` | `fred@dungeon.acme.com` |
| `userid@domain` | `fred@acme.com` |

- An **MX record (mail exchanger)** in the **recipient's DNS** gives the **IP address of the domain's email server**.

### 9. Extending email → MIME ★
- Email originally sent **short text notes** only. Now people send **graphics, audio, video, program binaries**; these show up as **"attached files."**
- **MIME (Multipurpose Internet Mail Extension):**
  - Allows **arbitrarily formatted data** to be sent via email.
  - Specifies the **content type** of the data: `text/html`, `image/gif`, `application/program`.
  - Can be used to include **security information**.

**MIME encoding**
- Data is sent as **printable characters** to avoid **translation and format errors**.
- **base64** encoding
- **quoted-printable** encoding: **7-bit ASCII**, with characters **above 127** sent as an **equal sign followed by two hex digits**.

### 10. Base64 (review from Lecture 19) ★
- Problem: binary data across networks that expect **regular characters**; some values (**return, comma, ETX, NAK, null**) confuse systems.
- **Base64** converts any data into a string of **alphanumeric characters**. ⚠️ **Encoding ≠ encryption — there is no key.**
- **3 bytes (24 bits) → four 6-bit values**; 2⁶ = **64** symbols = **26 upper + 26 lower + 10 digits + `+` and `/`**.
- Slide example: `X`, `CR`, `LF` → `010110 000000 110100 001010` → **`W A 0 K`**.
- **Padding with `=`** when input length isn't a multiple of 3:

| Input | Len | Output | Len | Padding |
|---|---|---|---|---|
| `light work.` | 11 | `bGlnaHQgd29yay4=` | 16 | **1** |
| `light work` | 10 | `bGlnaHQgd29yaw==` | 16 | **2** |
| `light wor` | 9 | `bGlnaHQgd29y` | 12 | **0** |
| `light wo` | 8 | `bGlnaHQgd28=` | 12 | **1** |
| `light w` | 7 | `bGlnaHQgdw==` | 12 | **2** |

- Example: `COMP476 Networked Computer Systems` → `Q09NUDQ3NiBOZXR3b3JrZWQgQ29tcHV0ZXIgU3lzdGVtcw==`
- Rule of thumb: a random-looking string of letters and numbers is **probably Base64**.
- *(Slide typo: says "256 eight-**byte** values" — it means 256 eight-**bit** values.)*

### 11. Remote mailbox access
- Some users access mail **remotely**; programs like **Outlook** and **Eudora** get the mail **from the mail server**.
- **Figure:** sending computer (mail transfer program) → **mail server** (on the computer with the mailbox) → **mailbox** → **POP server** → **POP client** on the user's computer.

### 12. SMTP ★
- Runs over **TCP**.
- Used **between:** the sender's computer and **their** email server, and **between mail servers**.
- Specifies **how:** the client interacts with the server, **recipients are specified**, the **message is transferred**.

**SMTP example (slide dialog)** — server replies in blue, client commands in green:
```
S: 220 ncatmail.ncat.edu ESMTP Sendmail ...
C: helo williams.comp.ncat.edu
S: 250 ncatmail.ncat.edu Hello williams.comp.ncat.edu [152.8.251.41], pleased to meet you
C: mail from: <williams@ncat.edu>
S: 250 <williams@ncat.edu>... Sender ok
C: rcpt to:<jb988330@ncat.edu>
S: 250 <jb988330@ncat.edu>... Recipient ok
C: data
S: 354 Enter mail, end with "." on a line by itself
C: This is an example of SMTP.
C: .
S: 250 RAA13983 Message accepted for delivery
C: quit
S: 221 ncatmail.ncat.edu closing connection
```
- **Command order to know:** `helo` → `mail from` → `rcpt to` → `data` → message → **`.` on a line by itself** → `quit`.
- **Reply codes:** **220** ready · **250** OK · **354** start mail input · **221** closing.

### 13. Email list servers
- Send mail to **many recipients**; users can usually **enroll or remove themselves**.
- Mail sent **to the list server** goes **to all users on the list**.
- Some are **moderated**.

### 14. Scenario: Alice sends a message to Bob ★ (step order is testable)
1. Alice uses her **user agent (UA)** to compose a message "to" `bob@someschool.edu`.
2. Alice's UA sends the message to **her mail server**; it is placed in a **message queue**.
3. The **client side of SMTP** opens a **TCP connection** with **Bob's mail server**.
4. The SMTP client sends Alice's message **over the TCP connection**.
5. Bob's mail server **places the message in Bob's mailbox**.
6. Bob invokes his **user agent** to read the message.

*(Slide credit: Dr. Jinsheng Xu.)*

### 15. Mail access protocols ★
- **SMTP:** delivery/storage **to the receiver's server**.
- **Mail access protocol:** **retrieval from the server**.
  - **POP** — Post Office Protocol **[RFC 1939]**: **authorization** (agent ↔ server) and **download**.
  - **IMAP** — Internet Mail Access Protocol **[RFC 1730]**: **more features (more complex)**; **manipulation of stored messages on the server**.
  - **HTTP** — Hotmail, Yahoo! Mail, etc. (webmail).

### 16. SMTP compared with HTTP ★
| SMTP | HTTP |
|---|---|
| Message (header **and** body) must be **7-bit ASCII** | |
| **Push** — server sends data when available | **Pull** — client requests the data |
| **Multiple objects** sent in a **multipart message** | **Each object** in its **own response message** |
| *Both:* **ASCII command/response** interaction, **status codes** | |

### 17. POP3 protocol ★
**Authorization phase**
- Client commands: **`user`** (declare username), **`pass`** (password).
- Server responses: **`+OK`**, **`-ERR`**.

**Transaction phase**
- **`list`** list message numbers · **`retr`** retrieve message by number · **`dele`** delete · **`quit`**.

```
S: +OK POP3 server ready
C: user bob
S: +OK
C: pass hungry
S: +OK user successfully logged on
C: list
S: 1 498
S: 2 912
S: .
C: retr 1
S: <message 1 contents>
S: .
C: dele 1
C: retr 2
S: <message 2 contents>
S: .
C: quit
S: +OK POP3 server signing off
```
*(The slide labels the second retrieval "message 1 contents" — a typo; it's message 2.)*

### 18. POP3 vs IMAP ★
| POP3 | IMAP |
|---|---|
| Example uses **"download and delete"** mode | Keeps **all messages in one place: the server** |
| Bob **cannot re-read** email if he **changes client** | Lets the user **organize messages in folders** |
| **"Download-and-keep"**: copies of messages on **different clients** | **Keeps user state across sessions** — folder names and mappings between **message IDs and folder names** |
| **Stateless across sessions** | |

---

## Part 2 — 🆕 FTP (File Transfer Protocol)

### 19. FTP basics ★
- **Transfers files to or from a remote host.** **Client/server** model:
  - **Client:** side that **initiates** the transfer (either to or from remote).
  - **Server:** the **remote host**.
- Defined by **multiple IETF RFCs**.
- Uses **TCP ports 20 and 21** *(slide typo: "TPC")*.
- Components: FTP user interface + FTP client (with **local file system**) ↔ FTP server (with **remote file system**).

### 20. Separate control and data connections ★
- Client contacts server at **TCP port 21** → **control connection**.
- Client gets **authorization** over the control connection and **browses the remote directory** by sending commands over it.
- When the server receives a **file transfer command**, it **opens a 2nd TCP connection** (for the file) **to the client** → **data connection, port 20**.
- After **one file**, the server **closes** the data connection; it opens **another** data connection for the next file.
- Control connection is **"out of band."**
- The FTP server **maintains state**: **current directory, earlier authentication**.

| Connection | Port | Purpose |
|---|---|---|
| Control | **21** | Login + commands (persistent) |
| Data | **20** | One file per connection |

### 21. NAT and firewall traversal ★
- The FTP server tries to create the second data channel by **opening a TCP connection to the client, from the server**.
- **NAT** and **firewalls** usually **block external (inbound) connections**.
- Fix: a **command makes the client create the other TCP connection to the server** instead. **Connections initiated from the Intranet can go through a NAT.**

### 22. Command line and window
- FTP was originally a **command-line program** (from the world **before GUIs**).
- **`ftp://` is no longer supported by web browsers.** (Slide screenshot: **FileZilla** GUI client.)

### 23. FTP commands and responses ★
| Command (ASCII over control channel) | Action |
|---|---|
| **`USER username`** | Send username |
| **`PASS password`** | Send password |
| **`LIST`** | Return list of files in current directory |
| **`RETR filename`** | **Retrieve (get)** a file |
| **`STOR filename`** | **Store (put)** a file onto the remote host |

| Return code | Meaning |
|---|---|
| **331** | Username OK, password required |
| **125** | Data connection already open; transfer starting |
| **425** | Can't open data connection |
| **452** | Error writing file |

*(Status code + phrase, **as in HTTP**.)*

---

## Part 3 — 🆕 Text Messaging

### 24. Text messages
- Originally = messages via **SMS (Short Message Service)**.
- Grown beyond alphanumeric text to include **MMS (Multimedia Messaging Service)** and **RCS (Rich Communication Services)**.

### 25. Texting volume
- About **23 billion** texts per day globally ≈ **270,000 per second**.
- Apps like **WhatsApp** and **Facebook Messenger** carry the **majority**, but **SMS remains in high use**.
- Side stat on the slide: the average American spends **5 h 24 min** on their mobile device each day.

### 26. Texting history
- Early **telegraph and radio** systems provided short text messaging.
- **Friedhelm Hillebrand** conceptualized SMS in **1984**; he noted most sentences were **160 characters or less**.
- First SMS: **3 December 1992**, **Neil Papworth** (22-year-old test engineer) sent **"Merry Christmas"** from a computer to a friend's phone.

### 27. SMS ★
- Runs on the **telephone system**; based on **Signaling System No. 7 (SS7)** — telephony signaling protocols **developed in the 1970s**, used to **set up and tear down calls** on most of the global **PSTN**.
- SMS sends data on the phone's **signaling channel** — the **same channel used for network control**.
- **Unused resources** are used to carry messages **at minimal cost**.

### 28. Control channel observations ★
- **SMS** sends data over the telephone **control channel**.
- **FTP** has a **separate channel for control**.
  - Two channels is **historic**, from very early protocols that were **simplex** (one direction only).
- **It is generally wise to separate data and control**; there are multiple ways to do it.

### 29. 7-bit character set
- SMS can use a variety of alphabets; the **default is a 7-bit character set** (good for English and a few other Western languages).
- **160 characters × 7 bits = 1,120 bits = 140 bytes** ✔ (matches the slide: "160 characters of 7 bits in 140 bytes").

### 30. SMS alternatives & RCS
- SMS is challenged by **IP-based messaging**: **Facebook Messenger, WhatsApp, Signal, Telegram, WeChat**. Usage varies by country.
- **RCS:** instant-messaging **protocol standard**, primarily for phones; a **replacement for SMS and MMS** on cellular networks; works between **Android and Apple iOS**; supports **end-to-end encryption**.

### 31. Short codes & message source
- **Short code** = a **five- or six-digit number** that can receive a text; often tied to **automated services**; leased by the short code program's registry service provider (**iconectiv**).
- Texts can come from **mobile devices** or **servers** (e.g., **appointment reminders**).

### 32. Satellite emergency texting
- New phones provide **emergency 911** contact **without cell connectivity**.
- Phone connects to a **satellite** (**geosynchronous or low earth orbit**) to send text.
- Some systems have you **fill out a form** that is transmitted to a help facility.

---

## Part 4 — 🆕 History aside: "Ancient Computing" (IBM VM/370)
- **1972:** IBM released **VM/370**. Each user got a **virtual machine** to boot whatever they wanted; most used **CMS (Conversational Monitor System)**. (The library at MTU booted **OS/MVT** for a catalog system that only ran on it.)
- **Virtual devices:** virtual printers (connected to real printers), virtual disks (a **portion of a real disk**); a virtual device could connect to a real one to **another user's virtual device**. Standard VMs still had a **virtual card reader and card punch** in the 1980s.
- **Email on VM/370:** the email program connected **your virtual card punch → recipient's virtual card reader**; it "punched" your email, which appeared in the recipient's reader; incoming files in the reader were displayed as incoming mail. **External email** went to a **communications virtual machine**.
- **Control Program (CP)** = the **hypervisor**; handles things outside your VM (e.g., connecting virtual devices). Its **`MSG` command** sent a **one-line message** to another user or the operator → **early texting**.

---

### 33. 🆕 Online API Assignment (reminder from L19)
- Three online APIs; details at **`comp476.online/APIasg/XXXX.pdf`**, **XXXX = your Base64-encoded magic number**.
- Due **noon, Wednesday, Nov 4, 2026**; upload source code to **Canvas**. *(Date has passed as of Nov 18.)*
- **Plagiarism slide:** Merriam-Webster definition (steal and pass off another's ideas/words as your own). The professor **assumes your program is your own**; **uncredited code that isn't yours = plagiarism.**

---

### 34. Quick fact sheet
| Concept | Key fact |
|---|---|
| Email is | An **ASCII file**: `name:value` header lines, **blank line**, body |
| Email model | **Store and forward**; background **daemon**; **spool directory** |
| Standards | **X.400** format · **RFC 822** text format · **SMTP (RFC 2821)** · **POP (RFC 1939)** · **IMAP (RFC 1730)** |
| SMTP | Runs over **TCP**; **7-bit ASCII**; **push** |
| SMTP command order | `helo` → `mail from` → `rcpt to` → `data` → `.` → `quit` |
| SMTP codes | **220** ready · **250** OK · **354** start data · **221** closing |
| Addressing | `userid@computer` or `userid@domain`; **MX record** in DNS → mail server IP |
| MIME | Content types (`text/html`, `image/gif`, `application/…`); **base64** or **quoted-printable** |
| Quoted-printable | 7-bit ASCII; chars >127 → **`=` + 2 hex digits** |
| Base64 | **3 bytes → 4 chars**, `=` padding, **not encryption** |
| POP3 | **Download-and-delete** (or keep); **stateless**; `user`/`pass`/`list`/`retr`/`dele`/`quit`; `+OK`/`-ERR` |
| IMAP | Mail **stays on server**; **folders**; keeps **state** across sessions |
| SMTP vs HTTP | **Push vs pull**; **multipart** vs **one object per response** |
| FTP ports | **21 control**, **20 data**; control is **out of band**; one data connection **per file** |
| FTP commands | `USER` `PASS` `LIST` `RETR` (get) `STOR` (put) |
| FTP + NAT | Server-initiated data connection gets **blocked** → client opens it instead |
| SMS | Over **SS7 signaling/control channel**; **160 chars × 7 bits = 140 bytes**; first text **Dec 3, 1992** |
| RCS | Replaces **SMS/MMS**; Android ↔ iOS; **end-to-end encryption** |
| Short code | **5–6 digit** number for automated services (**iconectiv**) |
| Separate data & control | FTP does it; SMS doesn't — **generally wise to separate** |
