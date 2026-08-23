# COMP 476 – Unit 3: Metric & Binary Prefixes

---

## Metric Prefixes

| Name | Symbol | Value |
|---|---|---|
| Yotta | Y | 10²⁴ |
| Zetta | Z | 10²¹ |
| Exa | E | 10¹⁸ |
| Peta | P | 10¹⁵ |
| Tera | T | 10¹² |
| Giga | G | 10⁹ |
| Mega | M | 10⁶ |
| Kilo | K | 10³ |
| milli | m | 10⁻³ |
| micro | μ | 10⁻⁶ |
| nano | n | 10⁻⁹ |
| pico | p | 10⁻¹² |
| femto | f | 10⁻¹⁵ |
| atto | a | 10⁻¹⁸ |
| zepto | z | 10⁻²¹ |
| yocto | y | 10⁻²⁴ |

The large prefixes (Kilo through Tera) are often used loosely for binary values too, since the powers of two land close to the corresponding powers of ten:

| Prefix | Power of 2 | ≈ Power of 10 | Exact Value |
|---|---|---|---|
| Tera (T) | 2⁴⁰ | about 10¹² | 1,099,511,627,776 |
| Giga (G) | 2³⁰ | about 10⁹ | 1,073,741,824 |
| Mega (M) | 2²⁰ | about 10⁶ | 1,048,576 |
| Kilo (K) | 2¹⁰ | about 10³ | 1,024 |

---

## SI Binary Prefixes (IEC 60027-2)

Because "kilo," "mega," etc. technically mean powers of *ten*, the International System of Units defines a **separate, unambiguous set of prefixes for binary (power-of-two) values.** These should be used whenever a value is truly a power of two (e.g., memory sizes, data rates) rather than reusing the metric prefixes.

| Name | Symbol | Value |
|---|---|---|
| yobi | Yi | 2⁸⁰ |
| zebi | Zi | 2⁷⁰ |
| exbi | Ei | 2⁶⁰ |
| pebi | Pi | 2⁵⁰ |
| tebi | Ti | 2⁴⁰ |
| gibi | Gi | 2³⁰ |
| mibi | Mi | 2²⁰ |
| kibi | Ki | 2¹⁰ |

> Example: 1 KiB (kibibyte) = 2¹⁰ = 1,024 bytes exactly, while 1 KB (kilobyte) technically means 1,000 bytes. In casual/industry use, "KB," "MB," "GB" are almost always used to mean the binary values (KiB, MiB, GiB) anyway — this is the source of a lot of confusion, especially in networking where data rates (Mbps, Gbps) are decimal but storage/memory sizes are usually binary.

---

## Key Terms
- **Metric (SI decimal) prefixes** — powers of 10, from yocto (10⁻²⁴) to yotta (10²⁴)
- **SI binary prefixes (IEC)** — powers of 2, from kibi (2¹⁰) to yobi (2⁸⁰); designed to remove ambiguity between decimal and binary "kilo/mega/giga," etc.
- **Kilo vs. Kibi** — 1,000 vs. 1,024; the source of most storage-size confusion
- **Further reading:** Hakan Temiz, "SI and Binary Prefixes: Clearing the Confusion,"
