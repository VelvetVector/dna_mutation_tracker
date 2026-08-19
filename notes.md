Genomic Mutation Tracker — Notes

0. Whole Project

                    generator.cpp
                         │
                         ▼
                  patients.txt
                         │
                         ▼
                    main.cpp
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
       ▼                 ▼                 ▼
    Load data           KMP           Rabin-Karp
       │                 │                 │
       │                 └──────┬──────────┘
       │                        │
       │                  Mutation results
       │                        │
       ▼                        ▼
 Patient information      Statistics / Risk
                                │
                                ▼
                       Global mutation frequency
                                │
                                ▼
                       Performance comparison
                                │
                                ▼
                           MySQL database

1. generator.cpp

Purpose

Generates synthetic patient DNA data and deliberately inserts known mutation patterns.

A. DNA Generation

Creates a random DNA sequence using the four bases:

A / T / C / G

B. Mutation Injection

Deliberately embeds predefined mutation patterns into the generated DNA.

Example:

Original DNA:
ACGTACGTACGT...

After injection:
ACGTACGGTACGT...
       ↑
    mutation

C. Age-Dependent Mutation Generation

The number of injected mutations increases with age.

Age ↑
  ↓
Expected injected mutations ↑

Important: This is only a simulation assumption used to generate data. It is not a medical/genomic fact.

2. KMP — Knuth-Morris-Pratt

Problem

Find all occurrences of one pattern inside a larger text efficiently.

In this project:

Text    → patient's DNA
Pattern → one known mutation

Core Idea

Naive matching may repeat comparisons after a mismatch.

KMP avoids this by preprocessing the pattern into an LPS array.

LPS

LPS[i] =
length of the longest proper prefix of pattern[0..i]
that is also a suffix of pattern[0..i]

The LPS array tells us:

After a mismatch, how much of the already-matched pattern can be reused?

KMP Flow

Step 1 — Build LPS

Pattern
   ↓
Build LPS array

Step 2 — Initialize

i = 0     → position in text
j = 0     → position in pattern

Step 3 — Search

              Compare text[i] and pattern[j]
                         │
              ┌──────────┴──────────┐
              │                     │
            MATCH               MISMATCH
              │                     │
              ▼                     ▼
          i++, j++                j > 0?
              │                  /      \
              │                YES       NO
              │                 │         │
              │                 ▼         ▼
              │            j=LPS[j-1]    i++
              │                 │         │
              └─────────────────┴─────────┘
                                │
                                ▼
                           Continue

When:

j == pattern length

the complete pattern has been found.

Then:

record position
      ↓
j = LPS[j-1]
      ↓
continue searching

This allows KMP to find overlapping occurrences as well.

KMP — What to Remember

1. What problem does KMP solve?

Efficiently searching for a pattern inside a larger string.

2. Why is naive matching inefficient?

After a mismatch, it may repeat comparisons that are already known to be unnecessary.

3. What is LPS?

The length of the longest proper prefix that is also a suffix for each prefix of the pattern.

4. What does this line do?

j = lps[j - 1];

It moves the pattern pointer to the next longest viable prefix, while the text pointer stays where it is.

5. Complexity

LPS construction = O(M)
Text scanning     = O(N)

Total             = O(N + M)

where:

N = text length
M = pattern length

3. Rabin-Karp

Problem

Find occurrences of multiple patterns inside the DNA sequence efficiently.

Unlike KMP, the project uses Rabin-Karp to handle multiple mutation patterns together.

A major issue is that the patterns can have different lengths.

For example:

GGT       → length 3
CGGA      → length 4
TAGG      → length 4
ATCGG     → length 5

A single fixed-size window cannot handle all of them.

Rabin-Karp Flow

                         DNA
                          │
                          ▼
                  List of patterns
                          │
                          ▼
                Group patterns by length
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Length 3     Length 4     Length 5
             │            │            │
             ▼            ▼            ▼
            GGT       CGGA, TAGG      ATCGG
             │            │            │
             ▼            ▼            ▼
        Hash patterns  Hash patterns  Hash patterns
             │            │            │
             ▼            ▼            ▼
        3-char windows  4-char windows  5-char windows
             │            │            │
             └────────────┼────────────┘
                          ▼
                    Compare hashes
                          │
                   Hash match?
                    /         \
                  NO           YES
                  │              │
                  ▼              ▼
              Move on       Compare actual
                             strings
                                │
                           ┌────┴────┐
                           ▼         ▼
                        Equal    Different
                           │         │
                           ▼         ▼
                         MATCH    Collision

How the hashing works

DNA has four possible characters:

A → 1
T → 2
C → 3
G → 4

The project uses base 4 for hashing.

For example:

ACG

becomes:

1 3 4

and its hash is calculated using positional values:

1 × 4² + 3 × 4¹ + 4 × 4⁰

Rolling Hash

Instead of recalculating the hash from scratch for every DNA window:

Window 1 → calculate hash
Window 2 → update previous hash
Window 3 → update previous hash
...

When the window moves:

Remove outgoing character
        ↓
Shift remaining characters
        ↓
Add incoming character
        ↓
New hash

This is called a rolling hash.

Why verify the actual string?

A hash match does not guarantee that the strings are equal.

Two different strings can theoretically produce the same hash:

Same hash
   ↓
Possible match
   ↓
Compare actual strings
   ↓
Equal? ── YES → Real match
   │
   NO
   ↓
Hash collision → Ignore

Therefore, the project uses the hash as a filter, followed by actual string comparison.

Rabin-Karp — What to Remember

1. Group patterns by length
2. Calculate hashes for patterns
3. Calculate hash of the first DNA window
4. Slide the window using rolling hash
5. Look up the window hash
6. If hash matches → compare actual strings
7. Store confirmed mutation positions

Core idea

KMP reuses previous character comparisons using LPS; Rabin-Karp avoids unnecessary string comparisons by using hashes and rolling them across the text.

4. Project-Level Mental Model

The three files fit together like this:

generator.cpp
      │
      │ generates synthetic DNA
      ▼
patients.txt
      │
      ▼
   main.cpp
      │
      ├───────────────┐
      ▼               ▼
     KMP          Rabin-Karp
      │               │
      │               │
 single pattern    multiple patterns
      │               │
      └───────┬───────┘
              ▼
       Mutation positions
              │
              ▼
       Patient statistics
              │
              ▼
       Risk / frequency analysis
              │
              ▼
          CSV / MySQL