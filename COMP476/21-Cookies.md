# COMP476 — Networked Computer Systems
## Lecture 21 — Cookies, Online Advertising, and the DOM (slides: "Cookies & Domain Object Model," Mon Nov 2, 2026)

**Relationship to earlier lectures:** this deck is the **application layer** again, building on **HTTP** (Lectures 18–19). HTTP is **stateless**; cookies are the mechanism that adds state on top. Items marked 🆕 are new. *(Slide title also lists COMP422 & COMP620 — shared deck.)*

---

## Part 1 — HTTP Cookies

### 1. What a cookie is ★
- A **small piece of data stored by the browser** on the client computer.
- Sent and received by the browser **with no interaction or knowledge of the user**.
- Cookies are **not displayed** by the browser.

### 2. Why cookies exist: stateless servers ★
- **Web servers and REST API servers are stateless.**
- **Each request is independent**, but the server *may* save information about all previous requests.
- **Cookies let the server connect saved information to a client on a second request.**
- Server programs can save **session information**, later located through information in the cookie.
- **Frequently a cookie holds only a database key** used to look up the user's information **on the server**.

### 3. Possible cookie contents
- **Authentication:** user name
- **Browser activity:** clicks, pages visited
- **Form data:** name, address, credit card number
- **Geographic location:** city, latitude and longitude
- …"and much, much more."

### 4. Session vs. persistent cookies ★
| | **Session cookie** | **Persistent cookie** |
|---|---|---|
| Stored | **Temporary memory** only, while the user navigates the site | **File system** on the client |
| Lifetime | **Deleted when the browser closes** | Has an **expiration date**, then deleted |
| Expiration date? | **No** | **Yes** (`Expires`), or **`Max-Age`** = seconds until expiry |
| Notes | May be **safer** — only the browser can access it | Called **tracking cookies** because advertisers use them to record browsing habits |

**Example header (from slide):**
```
Set-Cookie: insomnia_cookie=eyJp…3D; expires=Thu, 18-Feb-2023 18:17:15 GMT;
            Max-Age=3600; path=/; HttpOnly; Secure; Domain=.insomniacookies.com
```
- Cookie value looks like **Base64** (random-looking letters/numbers; see Lecture 19/20).

### 5. Cookie options ★
| Option | Meaning |
|---|---|
| **Secure** | Cookie only transmitted over an **encrypted connection (HTTPS)** |
| **HttpOnly** | Cookie **cannot be accessed by JavaScript** |
| **Path** | Where the cookie applies; often **`/`** (root of the site) |
| **Domain** | Cookie available to **all subdomains** of this domain. The domain **cannot be from a different domain than the source** of the cookie |
| **Expires / Max-Age** | Persistent-cookie lifetime (see §4) |

### 6. SameSite option ★
- Gives **some cross-site request forgery (CSRF) protection**.
- Values:

| Value | Behavior |
|---|---|
| **Strict** | Send the cookie **only to the same site that set it** |
| **Lax** | Not sent to other sites, **but sent when navigating to the original site from an external site** (e.g., **following a link**) |
| **None** | Sent with **both cross-site and same-site** requests |

---

## Part 2 — Third-Party Cookies and Tracking

### 7. First-party vs. third-party ★
- **First-party cookie:** from the site **in the address bar**.
- **Third-party cookie:** from a **different site**.
- A page may load images/text from **many other sites**, especially **advertising sites**.
- **Slide example:** you visit `www.acme.com`, which has ads from `adguy.com` → `adguy.com` sends a third-party cookie. Later you visit `www.other.com`, which **also** has ads from `adguy.com` → the browser **sends all `adguy.com` cookies**, so **adguy.com can track your browsing across both sites**.
- (Slide screenshot: a webpage that creates **many** third-party cookies.)

### 8. Targeting advertisements
- Look at a product on **Amazon** or another store → later see **ads for that product on other sites**.
- **Web advertising brokers** sell advertising services.
- Sites include a **link to the broker**, who sends **ad text and images targeted to the individual**.

### 9. Tracking cookies ★
- Used to **track browsing habits**.
- A third-party cookie server can keep a database of:
  - **pages and content** browsed
  - **what the user searched** online
  - **when the user clicked an ad**
  - **what time** the user visited a site

### 10. Web beacons 🆕
- **Embedded in web pages or email** to **report to someone that the page is being viewed**.
- The **Beacon API** sends an **asynchronous, non-blocking request that expects no response**.

---

## Part 3 — Uses of Cookies and Their Problems

### 11. Session management ★
- Cookies let a **stateless server** let a user **traverse many pages**, each showing **that user's** information.
- Cookies often contain a **unique session identifier** (a **long random string** of letters/numbers).
- When the user **logs in**, the server **remembers the session ID** of the authenticated user.

### 12. Personalization
- Store **how the user wants to interact** with pages from that server.
- Save info to **auto-complete form fields**.
- **Options, limits, and assorted data** can be kept in a cookie.
- **Slide example: NC Digital Library cookie** — a long cookie string holding things like **preferred library key, hold limit (6), checkout limits by media type, reading-history setting, recommendation limits**. (Shows a cookie can carry lots of readable app data, not just an ID.)

### 13. Viewing cookies ★
- **`curl`** shows returned headers and cookies.
- **Browser debugger** (Chrome: **Ctrl+Shift+I**): add a **watch for `document.cookie`**; mouse over it to see the value.
- An **HTTP proxy** will show cookies.

### 14. `document.cookie` ★
- A **string containing all the cookie information**.
- **Setting `document.cookie` to a value stores a cookie.**
- **Not available for HttpOnly cookies.**

### 15. Issues with cookies ★
- A cookie really identifies a **userid + computer + browser combination**.
- Switch **computers or browsers** → you **no longer have** the earlier cookies.
- **Back button** problem: after receiving a cookie then pressing back, the **page state and cookie state may be inconsistent**.

### 16. GDPR 🆕 ★
- **EU adopted the General Data Protection Regulation in 2016.**
- Natural persons may be associated with **cookie identifiers**, so **cookies can be personal data** and are **subject to GDPR**.
- To use such cookies, companies must get **prior user consent** (hence the cookie banners).

### 17. Browser privacy modes
- **Chrome Incognito:** doesn't permanently store **history, cookies, site data, or form inputs**.
- **Microsoft Edge: InPrivate** — works similarly.
- **Safari: Private Browsing** — same purpose.

---

## Part 4 — 🆕 Online Advertising and Click Fraud

### 18. Parties involved ★
| Party | Role |
|---|---|
| **Publisher** | **Website owner hosting the ad** |
| **Advertiser** | Entity that wants to **advertise its product** |
| **Ad Network** | **Mediator** between publisher and advertiser |

### 19. Ad servicing architecture ★ (step order)
*(Figures from an EPFL paper on online ad fraud.)*
1. User's browser **requests the web page**.
2. The page contains the **publisher's content** plus a **block of HTML from the Ad Network** that **redirects the browser to an ad server** (owned by the Ad Network). *Why:* keeping the ad's full code at the ad server **reduces the advertiser's coding burden** and gives the Ad Network **flexibility to change code easily**.
3. Browser **requests a script** (e.g., JavaScript) from the ad server.
4. Ad server **sends the script**. When run, it **collects parameters from the user's machine** that influence ad selection — **including HTTP cookies** the ad server deposited in earlier interactions.
5. Browser **executes the script** and sends the collected info **back to the ad server with the request for ads**.
6. Ad server **chooses and serves the most appropriate ads**; the browser **merges the ads with the page elements** already downloaded.
*(Slide numbers the last step "5" twice — it's step 6.)*

### 20. Advertisement revenue models ★
| Model | Advertiser is charged per… |
|---|---|
| **CPM** (cost per mille) | **Thousand views / impressions** |
| **CPC** (cost per click), a.k.a. **PPC** (pay per click) | **Click** |
| **CPA** (cost per action) | **Action**, e.g., an **online sale** |

### 21. Search results
- **Organic results:** appear due to **relevance** to the search term.
- **Sponsored results:** **ads** by advertisers **contracted with the search engine**.
- Engines try to show the **most relevant sponsored result** to tempt a click → **revenue**.
- **Both types are prone to ad fraud.**

### 22. Publisher ad fraud ★
- Generate **unwarranted revenue** with **human clickers or botnets**.
  - **Botnet:** a number of **internet-connected computers communicating with each other** to do a repetitive task, e.g., **clicking ads repeatedly**.
- **Manipulate keywords** on their sites to **attract higher-value ads**.
- Display ads **invisibly** (many ads "on the page" all generating revenue).

### 23. Advertiser ad fraud ★
- Hurt **rivals** using **human clickers or botnets** that **click the rival's ads**.
- Rival **pays for activity that never leads to a sale**.

### 24. Click fraud detection
- **Google's Ad Traffic Quality Team:** live reviewers, automatic filters, machine learning, deep research.
- **77%** of all internet exposures are carried out by **bots** (slide stat).
- **HUMAN** (fraud-detection company) verifies the humanity of **10+ trillion interactions per week**.

### 25. Defenses ★
| Publisher defenses | Advertiser defenses |
|---|---|
| **Site lists** managed by ad network providers can **prohibit an ad from being copied** to another site | Track **IP address**, **click timestamp** (user directed to their site), **action timestamp** (user acted on their site) |
| Prohibit **competitors' ads** on their site | **Many clicks from an IP with no action → likely fraud** |
| **Restrict ad content** (e.g., no drugs, sex, rock and roll) | **Exclude geographic regions** (click farms / likely fraud) |
| | **Remarketing:** show ads **only to users who previously visited** their site |

### 26. Case: Aleksandr Zhukov 🆕
- **Nov 10, 2021:** self-proclaimed **"King of Fraud"**, Russian **Aleksandr Zhukov**, **sentenced to 10 years** and fined **$3,827,493**.
- Ran a **large botnet that mimicked humans**.
- **Leased 765,000+ IP addresses**, assigned **multiple IPs to each datacenter server**, and **fraudulently registered IPs in the names of major U.S. internet providers with false location info**.

---

## Part 5 — 🆕 HTML Domain Object Model (DOM)

### 27. What the DOM is ★
- A **standard** defining the **objects and properties of HTML elements** and the **methods to access them**.
- **Object-oriented**, with **hierarchical nodes** for all elements and data in a web page.

### 28. Node properties ★
For a node `x`:
| Property | Meaning |
|---|---|
| `x.innerHTML` | **Text value** of x |
| `x.nodeName` | **Name** of x |
| `x.nodeValue` | **Value** of x |
| `x.parentNode` | **Parent** node |
| `x.childNodes` | **Child** nodes |
| `x.attributes` | **Attribute** nodes |

### 29. Finding a node by ID ★
- `document.getElementById(id)` finds the node with that `id`; put an `id` attribute on any element when writing the HTML.
```html
<p id="theOne">Original message</p>
<script>
var myParagraph = document.getElementById("theOne");
myParagraph.innerHTML = "Something completely new";
</script>
```
- ⚠️ **Slide typo:** it writes `getElementByID` — the real method is **`getElementById`** (lowercase **d**). JavaScript is case-sensitive.

### 30. Changing element style
- `x.style.property` changes a style property.
```html
<p id="id1">Normal stuff</p>
<button type="button"
  onclick="document.getElementById('id1').style.color = 'red'">Click Me!</button>
```

### 31. Changing an attribute's value
- `x.attributename` changes that attribute.
```html
<img id="myImage" src="sad.gif">
<script>
document.getElementById("myImage").src = "happy.jpg";
</script>
```

### 32. JavaScript functions and events
- JavaScript has many **built-in functions/methods**. **`alert("message")`** shows a **pop-up box** (popular in **hacking demos**).
- **`+` concatenates strings:** `alert("Hi " + user);`
- **Events** trigger JavaScript execution: **mouse click**, **mouse over an element**, **user enters a page**, **user leaves a page**, **element changed**.

---

### 33. Online API Assignment (reminder)
- Three online APIs; details at **`comp476.online/APIasg/XXXX.pdf`**, **XXXX = your Base64-encoded magic number**.
- Due **noon, Wednesday, Nov 4, 2026**; upload source code to **Canvas**. *(Date has passed as of Nov 18.)*

---

### 34. Quick fact sheet
| Concept | Key fact |
|---|---|
| Cookie | Small data stored by browser; sent/received **without user knowledge**; not displayed |
| Why | HTTP/REST servers are **stateless**; cookie links requests to **saved server-side info** (often just a **DB key**) |
| Session cookie | **Temp memory**, deleted on browser close, **no expiration date** |
| Persistent cookie | **File system**, has **Expires** or **Max-Age** (seconds); a.k.a. **tracking cookie** |
| Secure | **HTTPS only** |
| HttpOnly | **Not accessible to JavaScript** (also hidden from `document.cookie`) |
| Path / Domain | Path often `/`; Domain = **all subdomains**, can't be a foreign domain |
| SameSite | **Strict** = same site only · **Lax** = also on link navigation from external sites · **None** = cross-site + same-site · helps vs **CSRF** |
| First vs third party | First = **site in address bar**; third = **different site** (e.g., ad servers) |
| Cross-site tracking | Same third-party ad domain on many sites → its cookies are sent every time → **browsing tracked** |
| Session ID | **Long random string**; server remembers it on **login** |
| Viewing cookies | `curl`, debugger **`document.cookie`** (Ctrl+Shift+I), HTTP proxy |
| Cookie identity | **userid + computer + browser** |
| GDPR | **EU, 2016**; cookies can be **personal data** → **prior consent** |
| Privacy modes | Chrome **Incognito**, Edge **InPrivate**, Safari **Private Browsing** |
| Web beacon | Reports a page/email **was viewed**; **Beacon API** = async, no response |
| Ad parties | **Publisher · Advertiser · Ad Network** (ad server owned by Ad Network) |
| Revenue | **CPM** (per 1,000 views) · **CPC/PPC** (per click) · **CPA** (per action) |
| Fraud | **Publisher** (botnets, keyword manipulation, hidden ads) · **Advertiser** (clicking rivals' ads) |
| Advertiser defense | Track **IP + click/action timestamps**; many clicks, no action = fraud; **geo-exclusion**; **remarketing** |
| Zhukov case | **Nov 10, 2021**; **10 years**, **$3,827,493**; **765,000+ leased IPs** |
| DOM | **Hierarchical object model** of the page; `getElementById`, `innerHTML`, `.style`, `.src` |
