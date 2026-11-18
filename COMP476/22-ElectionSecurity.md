# COMP476 — Networked Computer Systems
## Lecture 22 — Election Security (slides: "Election Security," file dated Nov 2, 2026)

**Relationship to earlier lectures:** a **topic/case-study lecture** — no new protocol. It applies security thinking (secrecy, integrity, isolation, auditing, diversity of systems) to **voting systems**, using **Guilford County, NC** as the worked example. Items marked 🆕 are new topics. ★ = most likely exam-worthy.

---

## Part 1 — Requirements and Context

### 1. Voting in America ★
- Must be a **U.S. citizen**, meet your **state's residency requirements**, and be **18+**.
- Must be **registered**.
- **Your vote is secret** — **nobody, including election workers**, knows who you voted for.
- **Every vote counts equally.**
- **The goal for security is to meet these requirements** (secrecy + integrity + equal counting).

### 2. Instructor's involvement
- Instructor has worked as a **precinct election judge** for several years and on a **Multipartisan Assistance Team** (helping people in **assisted living facilities** vote).

### 3. Facts about election security in NC ★
*(From an NC State Board of Elections publication.)*
- **No evidence of successful attack**
- **Paper ballots**
- **Certified voting systems**
- **No network connections**
- **Logic and accuracy testing**
- **Trained officials**
- **Post-election audits**
- **Investigations division**

### 4. Many voting systems = defense through diversity ★
- **States and local jurisdictions** usually control voting.
- **~100,000 polling places** in the U.S.
- **~1 million people** involved in tabulating results.
- This diversity makes a **coordinated attack against all systems much harder.**

### 5. Computers in election tabulation ★
- Individual voting machines are **often isolated**.
- Results are **aggregated on county and state computer systems**.
- **There is no national vote aggregation system.**
- The **Electoral College representatives** decide the presidential election.
- **County and state computers could be vulnerable** (the weak point).

### 6. Guilford County, NC (numbers)
- **Charlie Collicutt** = Director, Guilford County Board of Elections.
- **134 polling places** in Guilford County.
- **7,554,158** NC voters in 2025 (slide says "eligible voters"; source is NCSBE registration stats, so likely **registered** voters).

---

## Part 2 — Registration and the Voting Process (Guilford County)

### 7. Registration
- Must register **before** you can vote; rules and process **vary across the country**.
- **NC:** register or cancel registration **by mail, at the Board of Elections, or at the DMV**.
- **Identification required:** name, address, date of birth, plus **NC driver license**, **last 4 of SSN**, or **state voter registration number**.

### 8. Same-day registration ★
- NC: you can **register and vote the same day during early voting**.
- You **cannot** register and vote on **Election Day**.

### 9. At the polling place — step order ★
1. Person at the table asks your **name and address**. **ID is now required.** **Misrepresenting yourself is a class 1 felony.**
2. They find your name in a **notebook (poll book)**.
3. They remove a **sticker with your name** and put it on an **Authorization To Vote (ATV)** form.
   - **Primaries:** an **unaffiliated (independent)** voter is asked **which party's ballot** they want.
4. You **sign the ATV**.
5. You give the ATV to **another election official**, who gives you a **paper ballot**.
6. You mark your choices at a **small private table** by **filling in the circle** next to the candidate.
7. You put the ballot into the **vote counting machine** (tabulation).

*(Why two officials: separates **identity checking** from **ballot issuing** — separation of duties.)*

### 10. Collecting information after polls close ★
- Results from the counting machine are written to a **USB flash memory card**.
- Machine prints **two paper copies** of the **aggregate count of all races**.
  - **Both signed by two election judges.**
  - **One copy** carried by the **chief judge** to county election HQ.
  - **Other copy** **mailed by a second judge.**
- **Paper ballots sealed in a box** and taken to county HQ.
- *(Redundancy: electronic card + two independent paper copies + the physical ballots.)*

### 11. Vote counting machine preparation ★
- Machines are **programmed by Guilford Board of Elections personnel — no outside contractor.**
- Machines must be told **what is on the ballots**.
- The **programming computers have no network connections** and are **stored in a vault** when not in use.
- A **password** is needed to start the counting machines.

---

## Part 3 — Auditing, Counting, and Results

### 12. Auditing ★
- A **statistically significant sample of precincts** is audited.
- Paper ballots are **re-run through a counting machine** and compared to the **reported numbers**.
- A **small random selection** of precincts is checked by **hand count**.
- **Machines are more accurate at ballot counting** than hand counts.

### 13. Hand-counting debate
- Some propose **hand-marked paper ballots be publicly hand-counted**.
- **Proponents:** **transparency** would reduce possible fraud.
- **Numbers from the slide:** **28 races** on this year's ballot; **285,595 ballots** cast in Guilford's **134 precincts in 2020** → a lot of hand-counting work (and machines are more accurate, per §12).

### 14. Official vs. press results ★
- Board gives the **press** a copy of results **as soon as available** from precincts.
- **Election-night results are unofficial.**
- **Official results** go to the **state within 10 days** of the election.

### 15. Poll observers ★
- **Public may not enter** a voting site to observe.
- Only observers **appointed in advance by a political party** may be inside.
- **May:** watch and **take notes (no photos)**; **report concerns to the chief judge**; get a **list of who has voted** at designated time intervals.
- **Must not:** **speak to voters**; **impede/disrupt** voting; **wear or distribute campaign material**; **position themselves to see the contents of voted ballots** (protects **secrecy**).
- Limit: **no more than 2 precinct-specific observers per party, plus 1 at-large observer.**

---

## Part 4 — Voting Methods

### 16. Old voting process (lever machines)
- Non-electric booth: **close the curtain with a large handle**, **push down levers** for selections, move the handle again → **adds votes to the total** and **opens the curtain**.
- An **array of mechanical counters** held the results.

### 17. Vote by mail ★
- Ballots mailed **~3 weeks before** the election.
- Voter marks ballot → puts it in a **secrecy envelope** → puts that in a **mailing envelope**.
- **Secrecy envelope** has a place for a **copy of photo ID or an ID Exception form**.
- Ballot must be **received by a specified date**.
- Ballots may be carried to **drop-off boxes**.
- *(Slides also show maps: **states allowing early voting**, and **change in voting methods** over time.)*

---

## Part 5 — 🆕 Foreign Interference

### 18. Foreign election intervention
- Most foreign-election interventions **1946–2000**: **United States — 81**, then **Russia — 36** (source: Wikipedia).
- **March 2021 declassified report:** **Russia's** interference aimed to **support Trump**; **Iran's** aimed to **hurt Trump**; **China** did **not** seek to influence the outcome.

### 19. Russian probing of state election systems ★
- Some states' systems were **scanned for vulnerabilities**.
- Some were **breached**, giving intruders access to **voting records**.
- **No election results were accessed or manipulated.**
- **Paul Manafort** (Trump's campaign manager) received **$60M+** from **pro-Russia political groups**; **sentenced to 7.5 years, served under two**.

---

## Part 6 — 🆕 Choosing Winners (Voting Theory)

### 20. Two-party system
- In most NC races, **only two candidates** run per position.
- Slide table (**candidates per race → number of races**): **1 → 13**, **2 → 22**, **3 → 3**, **4 → 1**, **9 → 1** (40 races total).

### 21. Runoff elections ★
- **Two candidates:** most votes wins.
- **More than two:** several systems exist. Some require **>50%**; if nobody gets it, a **runoff between the top two**.

### 22. Ranked voting systems
- Used in some other **states/countries** with many candidates.
- Voters **rank candidates in preference order** instead of choosing one favorite.
- **Several ways to determine the winner** (e.g., **ranked-choice voting**, **Condorcet**).

### 23. Condorcet method ★
- Devised by **Marquis de Condorcet** (18th-century French mathematician/philosopher).
- **Winner = candidate who beats every other candidate in a pairwise (head-to-head) election.**
- Usually works, but **in some situations there is no such winner** (cycles).

### 24. Condorcet example — Kentucky capital ★
*Voters always pick the closest city.*

| Voter group | 1st | 2nd | 3rd | 4th |
|---|---|---|---|---|
| **42%** (near Memphis) | Memphis | Nashville | Chattanooga | Knoxville |
| **26%** (near Nashville) | Nashville | Chattanooga | Knoxville | Memphis |
| **15%** (near Chattanooga) | Chattanooga | Knoxville | Nashville | Memphis |
| **17%** (near Knoxville) | Knoxville | Chattanooga | Nashville | Memphis |

| Pair | Winner |
|---|---|
| Memphis 42% vs Nashville 58% | Nashville |
| Memphis 42% vs Chattanooga 58% | Chattanooga |
| Memphis 42% vs Knoxville 58% | Knoxville |
| Nashville 68% vs Chattanooga 32% | Nashville |
| Nashville 68% vs Knoxville 32% | Nashville |
| Chattanooga 83% vs Knoxville 17% | Chattanooga |

- **Nashville beats all three others head-to-head → Nashville is the Condorcet winner** (even though **Memphis has the most first-choice votes at 42%**). *(The slide doesn't state the conclusion; it follows from the table.)*

---

## Part 7 — Course Housekeeping

### 25. Online API Assignment (reminder)
- Three online APIs; details at **`comp476.online/APIasg/XXXX.pdf`**, **XXXX = your Base64-encoded magic number**.
- Due **noon, Wednesday, Nov 4, 2026**; upload source code to **Canvas**. *(Date has passed as of Nov 18.)*

### 26. Good programs / plagiarism
- A good program: has **comments** (include a **comment at the top saying who wrote it**), **actually works**, **doesn't use deprecated methods**, and notes anything **unusual about how to run it** in comments.
- **Plagiarize** (Merriam-Webster): steal and pass off another's ideas or words as one's own, or use another's production **without crediting the source**.
- Instructor assumes your program is your own; **uncredited non-original parts = plagiarism.**

### 27. COMP476 online
- Classes on **Zoom** in addition to in person in **Graham 210** (Zoom ID **941 8575 9617**, also in syllabus).
- **Recorded lectures on YouTube**; links on **Canvas → Modules / videos**.
- **Monday, Nov 16, 2026 lecture: Zoom only** (no on-campus class). *(Date has passed as of Nov 18.)*

---

### 28. Quick fact sheet
| Concept | Key fact |
|---|---|
| Security goals | **Secret ballot** + **every vote counts equally** + only **registered, eligible** voters |
| NC protections | **Paper ballots**, **certified systems**, **no network connections**, **logic & accuracy testing**, **post-election audits** |
| Why attacks are hard | **~100,000 polling places**, **~1M tabulators**, **local control**; **no national aggregation system** |
| Weak point | **County and state aggregation computers** |
| Registration (NC) | Mail / Board of Elections / DMV; **same-day registration during early voting only** (not Election Day) |
| Check-in | Name + address → **poll book** → **ATV form** → **sign** → **paper ballot** (ID required; false ID = **class 1 felony**) |
| After close | **USB card** + **2 signed paper tapes** (one carried, one mailed) + **sealed ballot box** |
| Machine prep | **Board staff only, no contractor**; **air-gapped** computers in a **vault**; **password** to start |
| Audit | **Statistical sample** of precincts re-run through machine; **small random hand count**; machines more accurate |
| Results | Election night = **unofficial**; official to state within **10 days** |
| Observers | **Party-appointed only**; **no photos, no talking to voters, no seeing marked ballots**; max **2 per party + 1 at-large** |
| Vote by mail | Ballots ~**3 weeks** early; **secrecy envelope** + mailing envelope; ID copy or exception form |
| Foreign interference | **U.S. 81**, **Russia 36** (1946–2000); Russia **probed/breached** some state systems — **no results changed** |
| Runoff | If **no >50%**, **top two** face off |
| Condorcet | Winner **beats every other candidate pairwise**; may not exist; **Nashville** in the example |
